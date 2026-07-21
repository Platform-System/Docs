# Wallet Backend Flow

Tai lieu nay chot contract backend cho luong vi hien tai.

## Service ownership

- `Wallet.API`
  - source-of-truth cho `Wallet` va `WalletTransaction`
  - giu `balance`
  - giu lich su `topup`, `payment`
- `Payment.API`
  - chi lo tao payment link va xu ly webhook payment ngoai
- `Ordering.API`
  - lo tao order va chon cach thanh toan cho order
- `Identity.API`
  - sau `sync session` se dam bao user co vi

## Business rules da chot

- moi user co 1 vi
- vi duoc auto tao khi `POST /api/users/sync` thanh cong
- topup vi di qua `Payment.API`
- khi payment topup thanh cong, `Wallet.API` cong so du va ghi transaction `Topup`
- khi tra order bang vi, `Ordering.API` goi `Wallet.API` de tru tien
- neu vi khong du tien, order khong duoc thanh toan
- user phai tu chon thanh toan ngoai neu muon tiep tuc
- tam thoi khong co rut tien
- tam thoi khong cho doi sang `wallet` neu order da co external payment link pending

## Public API via Gateway

Gateway route:

- `/api/wallet/*` -> `Wallet.API`
- `/api/ordering/*` -> `Ordering.API`

### 1. Get current wallet

`GET /api/wallet/me`

Response:

```json
{
  "isSuccess": true,
  "value": {
    "id": "wallet-guid",
    "userId": "user-guid",
    "balance": 150000
  },
  "errors": []
}
```

### 2. Get wallet transactions

`GET /api/wallet/me/transactions?page=1&pageSize=10&type=1&status=2&referenceType=WALLET_TOPUP&referenceCode=250520123456&createdAtFrom=2026-05-20T00:00:00Z&createdAtTo=2026-05-20T23:59:59Z`

Response item:

```json
{
  "id": "transaction-guid",
  "walletId": "wallet-guid",
  "type": 1,
  "status": 2,
  "amount": 100000,
  "balanceAfter": 150000,
  "currency": "VND",
  "referenceType": "WALLET_TOPUP",
  "referenceId": "wallet-guid",
  "referenceCode": 250520123456,
  "description": "Wallet topup 250520123456",
  "createdAt": "2026-05-20T04:10:00Z"
}
```

Ghi chu:

- `type`: `1=Topup`, `2=Payment`, `3=Refund`
- `status`: `1=Pending`, `2=Succeeded`, `3=Failed`
- tat ca filter la optional
- `referenceType` nen dung gia tri business nhu `WALLET_TOPUP`, `ORDER`
- `createdAtFrom`, `createdAtTo` dung ISO-8601 UTC

### 2b. Admin get wallet by user id

`GET /api/wallet/users/{userId}`

Role:

- `admin`

### 2c. Admin get wallet transactions by user id

`GET /api/wallet/users/{userId}/transactions?page=1&pageSize=20&type=2&status=2&referenceType=ORDER&referenceCode=250520123456&createdAtFrom=2026-05-20T00:00:00Z&createdAtTo=2026-05-20T23:59:59Z`

Role:

- `admin`

Muc tieu:

- QA/backoffice co the tra balance va lich su giao dich cua bat ky user nao
- khong can query truc tiep DB de debug topup/payment flow

### 2d. Get current wallet statement

`GET /api/wallet/me/statement?createdAtFrom=2026-05-20T00:00:00Z&createdAtTo=2026-05-20T23:59:59Z`

Response:

```json
{
  "isSuccess": true,
  "value": {
    "walletId": "wallet-guid",
    "userId": "user-guid",
    "currentBalance": 110000,
    "openingBalance": 100000,
    "closingBalance": 110000,
    "totalTopup": 50000,
    "totalPayment": 40000,
    "netChange": 10000,
    "succeededTransactionCount": 2,
    "pendingTransactionCount": 1,
    "failedTransactionCount": 0,
    "createdAtFrom": "2026-05-20T00:00:00Z",
    "createdAtTo": "2026-05-20T23:59:59Z"
  },
  "errors": []
}
```

### 2e. Admin get wallet statement by user id

`GET /api/wallet/users/{userId}/statement?createdAtFrom=2026-05-20T00:00:00Z&createdAtTo=2026-05-20T23:59:59Z`

Role:

- `admin`

### 3. Create wallet topup

`POST /api/wallet/me/topups`

Request:

```json
{
  "amount": 100000,
  "provider": "PayOS"
}
```

Response:

```json
{
  "isSuccess": true,
  "value": {
    "walletId": "wallet-guid",
    "transactionId": "transaction-guid",
    "referenceCode": 250520123456,
    "amount": 100000,
    "currency": "VND",
    "provider": "PayOS",
    "checkoutUrl": "https://...",
    "paymentLinkId": "plink-123"
  },
  "errors": []
}
```

## Order payment flow

### 1. Create order from current cart

`POST /api/ordering/carts/checkout`

Request:

```json
{
  "recipientName": "Nguyen Van A",
  "phoneNumber": "0901234567",
  "city": "Ho Chi Minh",
  "district": "District 7",
  "ward": "Tan Phu",
  "streetAddress": "123 Nguyen Luong Bang",
  "shipmentMethod": "Standard",
  "shipmentFee": 0,
  "shipmentNote": "Call before delivery"
}
```

Response co `orderCode` de dung cho buoc thanh toan.

### 2. Pay order with wallet

`POST /api/ordering/orders/{orderCode}/checkout`

Request:

```json
{
  "paymentMethod": "wallet"
}
```

Ket qua:

- neu du tien: order `Paid`, payment snapshot `Paid`
- neu khong du tien: `400` voi message `The wallet balance is insufficient.`

### 3. Pay order with external payment

`POST /api/ordering/orders/{orderCode}/checkout`

Request:

```json
{
  "paymentMethod": "payment",
  "provider": "PayOS"
}
```

Ket qua:

- tra `checkoutUrl`
- `Ordering.API` luu pending payment snapshot
- `Payment.API` xu ly webhook va publish `PaymentSucceeded`
- `Ordering.API` nhan event roi mark order paid

## Internal integrations

### Identity -> Wallet

- `Identity.API` publish Kafka event `IdentityUserSynced`
- `Wallet.API` consume event va tao vi neu user chua co vi

### Ordering -> Wallet

- `Ordering.API` goi gRPC `PayOrder`
- chi dung khi `paymentMethod = wallet`

### Payment -> Wallet

- `Payment.API` publish event `PaymentSucceeded`
- `Wallet.API` consumer event neu `ReferenceType = WALLET_TOPUP`
- consumer cong so du va mark pending topup transaction thanh `Succeeded`

## Files canary

Neu can tim nhanh code chinh:

- `Wallet.API/Presentation/Http/WalletsController.cs`
- `Wallet.API/Application/Features/Wallets/Commands/CreateTopup/CreateWalletTopupHandler.cs`
- `Wallet.API/Consumers/PaymentSucceededConsumer.cs`
- `Ordering.API/Application/Features/Orders/Commands/Checkout/CheckoutOrderHandler.cs`
- `Identity.API/Application/Features/Users/Commands/SyncUserSession/SyncUserSessionHandler.cs`
