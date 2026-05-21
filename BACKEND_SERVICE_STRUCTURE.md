# Backend Service Structure

Tài liệu này chốt "bản đồ src" cho các backend service trong hệ `Platform.*.API`.

Mục tiêu:

- nhìn folder là biết file nên nằm ở đâu
- tạo feature mới mà không bị đặt sai tầng
- giữ style đồng nhất giữa các service

## Cây thư mục chuẩn

```text
Platform.<Service>.API
├─ Application
│  ├─ Abstractions
│  │  ├─ Integrations
│  │  ├─ Messaging
│  │  └─ <OtherContracts>
│  └─ Features
│     └─ <FeatureName>
│        ├─ Commands
│        ├─ Queries
│        ├─ Responses
│        ├─ Mappers
│        └─ Services
├─ Domain
│  ├─ Entities
│  ├─ Enums
│  ├─ Errors
│  ├─ Events
│  └─ ValueObjects
├─ Infrastructure
│  ├─ Configurations
│  ├─ Constants
│  ├─ Data
│  │  ├─ <Service>DbContext.cs
│  │  ├─ <Service>DbContextFactory.cs
│  │  └─ Migrations
│  ├─ DependencyInjection
│  │  └─ DependencyInjection.cs
│  ├─ Integrations
│  ├─ Persistence
│  │  ├─ Configurations
│  │  └─ Models
│  ├─ Providers
│  ├─ Outbox
│  └─ Services
├─ Presentation
│  ├─ Http
│  └─ Grpc
├─ Consumers
├─ Properties
├─ Protos
├─ Program.cs
├─ appsettings.json
├─ appsettings.Development.json
├─ Platform.<Service>.API.csproj
└─ Platform.<Service>.API.Tests
   ├─ Application
   ├─ Infrastructure
   ├─ Presentation
   └─ Consumers
```

## Ý nghĩa từng tầng

- `Application`
  - chứa use case
  - nơi đặt `Command`, `Query`, `Handler`, `Validator`, `Response`, `Mapper`
  - nếu muốn biết service "làm gì" thì tìm ở đây

- `Domain`
  - chứa business core
  - nơi đặt `Entity`, `Enum`, `Error`, `ValueObject`, `Domain Event`
  - không phụ thuộc HTTP, DB, gRPC

- `Infrastructure`
  - chứa DB và kết nối bên ngoài
  - nơi đặt `DbContext`, migration, EF model/configuration, options, gRPC/HTTP client, provider, outbox

- `Presentation`
  - chứa HTTP controller và gRPC service
  - nên mỏng
  - nhận request, gọi command/query, trả response

- `Consumers`
  - chứa message consumer
  - xem như một entrypoint của service, ngang hàng với `Presentation`
  - convention chốt là đặt `Consumers/` ở root service, không ưu tiên `Infrastructure/Consumers`

- `Program.cs`
  - entry point bootstrap service
  - chỉ nên chứa wiring như:
    - `AddApplication(...)`
    - `Add<Service>Infrastructure(...)`
    - auth, runtime, swagger
    - `ApplyMigrationsAsync<TDbContext>()`
    - `MapControllers()`, `MapGrpcService(...)`

## Đặt file vào đâu

- thêm API mới
  - `Presentation/Http`
  - và logic ở `Application/Features/<FeatureName>`

- thêm gRPC endpoint mới
  - `Presentation/Grpc`

- thêm business rule, entity, error nghiệp vụ
  - `Domain`

- thêm `DbContext`, migration, EF model, EF configuration
  - `Infrastructure/Data`
  - `Infrastructure/Persistence/Models`
  - `Infrastructure/Persistence/Configurations`

- thêm config options
  - `Infrastructure/Configurations`

- thêm client gọi service khác
  - `Infrastructure/Integrations`

- thêm payment provider / sandbox provider / strategy
  - `Infrastructure/Providers`

- thêm outbox dispatcher / outbox writer
  - `Infrastructure/Outbox`

- thêm consumer RabbitMQ / MassTransit
  - `Consumers`

## Rule nhớ nhanh

- `Presentation` = cửa vào
- `Application` = xử lý use case
- `Domain` = luật nghiệp vụ cốt lõi
- `Infrastructure` = chạm DB và bên ngoài
- `Consumers` = cửa vào async

## Notes

- Khác biệt do domain là bình thường.
  - ví dụ service có `Providers` hoặc `Outbox` không có nghĩa là lệch style
- Lệch style chỉ xảy ra khi đặt sai tầng.
  - ví dụ business logic dày trong controller
  - hoặc migration nằm sai chỗ
  - hoặc consumer bị chôn vào folder không rõ vai trò
