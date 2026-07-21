# Backend Service Structure

Tài liệu này chốt "bản đồ src" cho các backend service trong hệ `*.API`.

Mục tiêu:

- nhìn folder là biết file nên nằm ở đâu
- tạo feature mới mà không bị đặt sai tầng
- giữ style đồng nhất giữa các service

## Sơ đồ nhớ nhanh

```text
<Service>.API
├─ Application      -> Use case
├─ Domain           -> Business core
├─ Infrastructure   -> DB + external systems
├─ Presentation     -> HTTP/gRPC entrypoint
├─ Consumers        -> Async message entrypoint
└─ Program.cs       -> Bootstrap
```

## Cây thư mục chuẩn

```text
<Service>.API
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
├─ <Service>.API.csproj
└─ <Service>.API.Tests
   ├─ Application
   ├─ Infrastructure
   ├─ Presentation
   └─ Consumers
```

## Đọc cây này như thế nào

- `Presentation` là cửa vào đồng bộ
  - nhận HTTP request hoặc gRPC call
  - không nên chứa business logic dày
  - `gRPC` vẫn nằm trong `Presentation/Grpc`, không kéo ra root

- `Consumers` là cửa vào bất đồng bộ
  - nhận message từ RabbitMQ, MassTransit, event bus
  - được xem như một entrypoint riêng của service

- `Application` là luồng xử lý use case
  - command, query, handler, validator, response, mapper

- `Domain` là lõi nghiệp vụ
  - entity, enum, error, value object, domain event

- `Infrastructure` là tầng chạm bên ngoài
  - database, config, integration client, provider, outbox

- `Program.cs` chỉ để bootstrap
  - đăng ký DI, middleware, migration, map endpoint

## Ý nghĩa từng tầng chi tiết

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

## Muốn thêm file gì thì đặt ở đâu

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

## Mapping nhanh theo loại file

```text
*Controller.cs                  -> Presentation/Http
*IntegrationService.cs          -> Presentation/Grpc
*Command.cs                     -> Application/Features/<Feature>/Commands
*Handler.cs                     -> Application/Features/<Feature>/Commands hoặc Queries
*Validator.cs                   -> Application/Features/<Feature>/Commands
*Query.cs                       -> Application/Features/<Feature>/Queries
*Response.cs                    -> Application/Features/<Feature>/Responses
*Mapper.cs                      -> Application/Features/<Feature>/Mappers hoặc Presentation/Grpc
<Service>DbContext.cs           -> Infrastructure/Data
<Service>DbContextFactory.cs    -> Infrastructure/Data
<Entity>Model.cs                -> Infrastructure/Persistence/Models
<Entity>Configuration.cs        -> Infrastructure/Persistence/Configurations
<Options>.cs                    -> Infrastructure/Configurations
<ServiceClient>.cs              -> Infrastructure/Integrations
<Provider>.cs                   -> Infrastructure/Providers
<EventConsumer>.cs              -> Consumers
```

## Rule nhớ nhanh

- `Presentation` = cửa vào
- `Application` = xử lý use case
- `Domain` = luật nghiệp vụ cốt lõi
- `Infrastructure` = chạm DB và bên ngoài
- `Consumers` = cửa vào async

## Chốt convention cho `Consumers`

- Ưu tiên `Consumers/` ở root service
- Không ưu tiên `Infrastructure/Consumers`
- Không kéo `Grpc/` ra root service
- Lý do:
  - `Consumers` là entrypoint của service
  - vai trò của nó gần với `Presentation` hơn là một implementation detail
  - nhìn cây thư mục sẽ dễ hình dung service có mấy cửa vào
  - `gRPC` vẫn là synchronous interface nên thuộc `Presentation`

## Notes

- Khác biệt do domain là bình thường.
  - ví dụ service có `Providers` hoặc `Outbox` không có nghĩa là lệch style
- Lệch style chỉ xảy ra khi đặt sai tầng.
  - ví dụ business logic dày trong controller
  - hoặc migration nằm sai chỗ
  - hoặc consumer bị chôn vào folder không rõ vai trò
