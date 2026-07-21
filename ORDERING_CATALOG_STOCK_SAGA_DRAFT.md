# Ordering Catalog Stock Saga Draft

Tài liệu này là bản nháp để chuyển flow orchestration stock giữa `Ordering.API` và `Catalog.API` từ `gRPC sync` sang `Kafka + Saga`.

## 1. Current Flow

Flow hiện tại:

1. user checkout cart
2. `Ordering.API` gọi gRPC sang `Catalog.API` để `DecreaseStock`
3. nếu trừ kho thành công thì `Ordering.API` tạo order local
4. user checkout order
5. `Ordering.API` gọi `Payment.API` để tạo payment link hoặc gọi `Wallet.API` để trừ tiền
6. `Payment.API` publish `PaymentSucceeded` hoặc `PaymentCancelled`
7. `Ordering.API` consume payment event
8. nếu payment bị hủy thì `Ordering.API` gọi gRPC sang `Catalog.API` để `RestoreStock`

Code canary:

- `Ordering.API/Application/Features/Carts/Commands/Checkout/CheckoutCartHandler.cs`
- `Ordering.API/Infrastructure/Integrations/Catalog/CatalogClient.cs`
- `Ordering.API/Consumers/PaymentCancelledConsumer.cs`
- `Ordering.API/Consumers/PaymentSucceededConsumerService.cs`

## 2. Pain Points

Những điểm cảnh báo của flow hiện tại:

- `Ordering.API` bị couple chặt với `Catalog.API` trong bài toán stock
- `Ordering.API` phải tự orchestration logic bù stock
- flow payment đã event-driven một phần, nhưng flow stock vẫn sync
- nếu `Catalog.API` chậm hoặc unavailable thì checkout cart bị ảnh hưởng trực tiếp
- compensation hiện tại nằm rải rác trong handler/consumer thay vì thành một saga rõ ràng

## 3. Saga Goal

Mục tiêu của bản mới:

- `Ordering.API` không gọi sync sang `Catalog.API` để trừ/trả stock
- stock được reserve/release bằng event
- order đi qua state machine rõ ràng
- payment fail hoặc cancel có compensation minh bạch
- mỗi bước có thể retry/idempotent/độc lập

## 4. Proposed Boundary

Boundary đề xuất:

- `Ordering.API`
  - source-of-truth cho order lifecycle
  - publish request-like event liên quan đến stock reservation
  - consume kết quả reserve/release để cập nhật order state

- `Catalog.API`
  - source-of-truth cho stock
  - consume event reserve/release stock
  - publish kết quả reserve/release

- `Payment.API`
  - tiếp tục giữ vai trò source-of-truth cho external payment status
  - vẫn publish `PaymentSucceeded` / `PaymentCancelled`

## 5. Proposed Order States

`Order` cần nhắc thêm các state sau:

- `PendingStockReservation`
- `PendingPayment`
- `Paid`
- `Cancelled`
- `StockReservationFailed`

Ý nghĩa:

- `PendingStockReservation`: order đã được tạo nhưng chưa có kết quả reserve stock
- `PendingPayment`: stock đã reserve xong, sẵn sàng thanh toán
- `StockReservationFailed`: reserve stock thất bại, order không thể đi tiếp

## 6. Proposed Events

Draft event đầu tiên:

1. `StockReservationRequested`
2. `StockReserved`
3. `StockReservationFailed`
4. `StockReleaseRequested`
5. `StockReleased`

Field tối thiểu nên có:

- `MessageId`
- `OrderId`
- `OccurredAt`
- `Items`
- `Reason` với failure/cancel flow
- `ReservationId` nếu muốn theo dõi vòng đời reservation riêng

## 7. Proposed Saga Flow

### Step A. Checkout Cart

1. user checkout cart
2. `Ordering.API` tạo order local với state `PendingStockReservation`
3. `Ordering.API` ghi outbox event `StockReservationRequested`
4. outbox dispatcher publish event lên Kafka

### Step B. Stock Reservation

1. `Catalog.API` consume `StockReservationRequested`
2. `Catalog.API` validate và reserve stock
3. nếu reserve thành công:
   - publish `StockReserved`
4. nếu reserve thất bại:
   - publish `StockReservationFailed`

### Step C. Ordering Receives Stock Result

1. `Ordering.API` consume `StockReserved`
2. đổi state order sang `PendingPayment`
3. lúc này UI mới nên cho thanh toán

Hoặc:

1. `Ordering.API` consume `StockReservationFailed`
2. đổi state order sang `StockReservationFailed` hoặc `Cancelled`
3. thông báo user hết hàng / không đủ stock

### Step D. Payment

Phần này chưa cần đổi transport:

- `Ordering.API -> Payment.API` tạo payment link vẫn là sync
- `Ordering.API -> Wallet.API` pay order vẫn là sync

### Step E. Compensation

Nếu payment fail, expire, hoặc user cancel:

1. `Ordering.API` publish `StockReleaseRequested`
2. `Catalog.API` consume event và release stock
3. `Catalog.API` publish `StockReleased`
4. `Ordering.API` consume `StockReleased` và mark order `Cancelled`

## 8. Why This Is A Saga

Flow này là saga vì:

- mỗi service tự commit transaction local của nó
- không có distributed transaction xuyên service
- bước sau fail thì có compensation
- coordination được làm bằng event + state machine

Compensation chính:

- đã reserve stock thì phải release stock khi payment fail/cancel

## 9. Command-like Events vs Domain Events

Trong draft này:

- `StockReservationRequested`
- `StockReleaseRequested`

có tính chất giống command được gửi qua event bus.

Còn:

- `StockReserved`
- `StockReservationFailed`
- `StockReleased`

là event xác nhận trạng thái đã xảy ra.

Điều này chấp nhận được nếu team muốn choreography nhẹ, chưa cần một orchestrator trung tâm riêng.

## 10. Risks Introduced By The New Design

Nếu đổi sang saga/Kafka, mình sẽ thêm độ phức tạp:

- thêm state trung gian cho order
- UI/API phải chấp nhận eventual consistency
- cần idempotency cho reserve/release
- cần inbox/outbox/retry/DLT
- cần xử lý event đến trễ hoặc duplicate
- cần reconciliation nếu event chain bị sót

## 11. Suggested Learning Path

Nếu mục tiêu là học thay vì đổi production ngay:

1. chốt state machine trên giấy
2. chốt event contract draft
3. chốt producer/consumer ownership
4. viết test case cho success/failure/duplicate/timeout
5. chỉ sau đó mới prototype code

## 12. Next Draft

Bước tiếp theo nên làm:

1. draft event contract theo style `Contracts`
2. draft state transition table cho `Order`
3. draft ownership của từng consumer/producer
