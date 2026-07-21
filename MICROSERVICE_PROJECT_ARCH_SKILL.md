# Microservice Project Arch Skill Backup

Tai lieu nay la ban backup trong repo cho local Codex skill:

- `microservice-project-arch`

Muc dich:

- tranh mat skill khi hong may hoac mat thu muc local `.codex`
- giu mot `source of truth` nam trong `Docs`
- de sau nay co the copy lai vao local skill folder ma khong can viet lai

## Local skill path

Thong thuong skill dang nam tai:

```text
C:\Users\<your-user>\.codex\skills\microservice-project-arch\SKILL.md
```

Voi may hien tai, path dang dung la:

```text
C:\Users\Khanh Hung\.codex\skills\microservice-project-arch\SKILL.md
```

## Cach khoi phuc skill sau nay

Co 2 cach:

### Cach 1. Dung script restore

Tu workspace root:

```powershell
./Workspace/scripts/restore-microservice-arch-skill.ps1
```

Neu can restore cho user khac:

```powershell
./Workspace/scripts/restore-microservice-arch-skill.ps1 -TargetUser "<your-user>"
```

Neu local skill da ton tai va muon ghi de:

```powershell
./Workspace/scripts/restore-microservice-arch-skill.ps1 -Force
```

### Cach 2. Copy tay

1. clone hoac mo lai workspace/repo
2. mo file backup nay trong `Docs`
3. copy toan bo noi dung skill ben duoi
4. ghi de vao local file:
   - `C:\Users\<your-user>\.codex\skills\microservice-project-arch\SKILL.md`

Script restore phia tren se doc dung khoi code ` ```md ` trong file nay va ghi ra local skill folder.

---

```md
---
name: microservice-project-arch
description: Áp dụng rule kiến trúc cho repo MicroServiceProject .NET backend. Use when editing Catalog.API, Ordering.API, Store.API, Identity.API, Payment.API, Wallet.API, BuildingBlocks, SystemContext, hoặc service liên quan để giữ đúng query -> model -> response, command -> domain -> response, service boundary, mapper/handler/folder structure, integration abstraction, và EF migration conventions.
---

# Microservice Project Arch

Skill này là rule tổng hợp cho backend service trong repo `MicroServiceProject`.

Phần này không chỉ cover application flow, mà còn cover:

- service boundary
- feature folder structure
- mapping/response/validation convention
- integration abstraction
- persistence và EF migration convention

## 1. Rule cốt lõi

- `query -> model -> response`
- `command -> domain -> response`
- handler chỉ orchestration
- handler không tự kéo rule domain/error message lên application response
- nếu có rule chéo subdomain/service, tạo abstraction trước rồi mới đổi local implementation sang gRPC hoặc integration client

## 2. Service boundary rule

- mỗi service là source-of-truth cho data và business rule của nó
- service khác không query trực tiếp persistence của service đó
- cross-service access phải đi qua abstraction, integration endpoint, hoặc client rõ ràng
- không để service A đọc bảng/model của service B chỉ vì cùng nằm trong workspace

Boundary mong muốn hiện tại:

- `Catalog`: product, category, stock
- `Store`: store profile, membership, activation status, store policy
- `Ordering`: cart, order, payment snapshot phục vụ checkout và history
- `Identity`: user profile nội bộ và identity-related integration surface
- `Payment`: payment link, payment transaction, payment webhook/outbox
- `Wallet`: wallet balance, wallet transaction, topup/payment ledger

## 3. Query rule

- query đọc từ `model`
- query map thẳng `model -> response`
- query không đi vòng qua domain nếu chỉ là read-only data
- nếu query cần đọc rule/dữ liệu chéo feature nhiều lần, ưu tiên read abstraction/service helper

## 4. Command rule

- command đi qua `domain`
- validate business rule ở domain hoặc policy service
- validate format/basic input ở validator
- xong mới map về persistence/response
- không để command thao tác thẳng model bỏ qua domain

## 5. Mapper rule

- `PersistenceMapper`: `domain <-> model`
- `ResponseMapper`: `domain -> response` hoặc `model -> response` với query
- `IntegrationMapper`: map request/response khi đi qua gRPC hoặc integration boundary
- không map tay dài trong handler nếu đã có mapper phù hợp

## 6. Handler rule

- handler chỉ điều phối
- không nhét business rule dài vào handler nếu rule đó thuộc domain/subdomain khác
- không query trực tiếp subdomain/service khác nếu đã có abstraction phù hợp
- lỗi trả về application nên ưu tiên `Result<T>.Failure(statusCode, message)`

## 7. Feature folder rule

Mỗi feature nên ưu tiên cấu trúc:

```text
Application/
  Features/<FeatureName>/
    Commands/
    Queries/
    Mappers/
    Responses/
    Services/
```

Ý nghĩa:

- `Commands` chứa `Command`, `Handler`, `Request`, `Validator`
- `Queries` chứa `Query`, `Handler`
- `Mappers` chứa `PersistenceMapper`, `ResponseMapper`, `IntegrationMapper`
- `Responses` chứa DTO trả ra API/application
- `Services` chỉ dùng khi feature cần read/service helper nội bộ rõ ràng

Không nên:

- đặt class của feature này trong folder của feature khác
- giữ `Shared` cho response mới nếu có thể đặt vào `Responses`

## 8. Response DTO rule

- response DTO dùng `sealed class`
- ưu tiên `init` cho field read-only response
- đặt trong folder `Responses`
- tên kết thúc bằng `Response`

Ví dụ:

- `StoreDetailsResponse`
- `UserResponse`
- `CurrentUserResponse`

Pattern cũ cần hạn chế mở rộng:

- `Features/<Feature>/Shared/<Something>Response.cs`

## 9. Validation rule

- command có input request thì ưu tiên có `AbstractValidator<TCommand>`
- validation format/basic rule nằm ở validator
- business rule nằm ở domain hoặc policy service
- không validate input thủ công trong handler nếu đã có validator

## 10. Controller và route rule

- controller dùng tên số nhiều khi đại diện resource:
  - `UsersController`
  - `ProductsController`
  - `StoresController`
  - `OrdersController`
- route ưu tiên explicit thay vì phụ thuộc `[controller]` nếu đang chốt API public lâu dài
- tách rõ public surface và manage surface nếu khác audience

Ví dụ:

- public: `api/stores`
- manage: `api/manage/stores`

## 11. Integration rule

- service source-of-truth cung cấp abstraction hoặc integration endpoint rõ ràng
- gRPC/controller integration nên map qua helper riêng:
  - `...IntegrationMapper`
  - `...IntegrationResponses`
- service gọi sang service khác thì dùng abstraction để dễ đổi local implementation sang gRPC client
- nếu hiện tại đang gọi local service helper nhưng boundary đã rõ, thiết kế abstraction trước để sau đó đổi implementation sang gRPC không phá application flow

## 12. Persistence và EF convention

Mỗi service có database riêng sẽ theo cấu trúc:

```text
<Service>.API/
  Infrastructure/
    Data/
      <Service>DbContext.cs
      <Service>DbContextFactory.cs
      Migrations/
        <timestamp>_<MigrationName>.cs
        <timestamp>_<MigrationName>.Designer.cs
        <Service>DbContextModelSnapshot.cs
    Persistence/
      Configurations/
      Models/
```

Ý nghĩa:

- `Data/` chứa EF bootstrap:
  - `DbContext`
  - `IDesignTimeDbContextFactory`
  - `Migrations`
- `Persistence/` chứa persistence shape:
  - `Models`
  - `EntityTypeConfiguration`

Namespace mong muốn:

- `DbContext`: `<Service>.API.Infrastructure.Data`
- `DbContextFactory`: `<Service>.API.Infrastructure.Data`
- `Migrations`: `<Service>.API.Infrastructure.Data.Migrations`
- `Models`: `<Service>.API.Infrastructure.Persistence.Models`
- `Configurations`: `<Service>.API.Infrastructure.Persistence.Configurations`

Không trộn:

- `Infrastructure.Data.Migrations`
- `Infrastructure.Persistence.Migrations`

Trong cùng một service chỉ được có một chuẩn duy nhất:

- `Infrastructure.Data.Migrations`

## 13. DbContext rule

- mỗi service có database riêng phải có đúng 1 `DbContext` chính
- `DbContext` đặt trong `Infrastructure/Data`
- `DbContext` map tới persistence model trong `Infrastructure/Persistence/Models`
- `OnModelCreating` apply configuration từ assembly hiện tại
- mỗi service có `DbContext` phải có `IDesignTimeDbContextFactory<TContext>` đặt cùng folder với `DbContext`

## 14. Migration rule

- migration phải nằm dưới `Infrastructure/Data/Migrations`
- không đặt migration trong `Persistence`
- snapshot phải đi cùng migration
- không edit tay `Designer.cs` nếu không thật sự cần rescue conflict
- migration name phải mô tả rõ thay đổi, ví dụ:
  - `InitialCreate`
  - `AddPaymentOutbox`
  - `AddStoreMedia`

Không nên đặt:

- `UpdateDb`
- `Fix`
- `TestMigration`

## 15. Naming rule

- role name dùng một chuẩn duy nhất trong toàn repo
- ưu tiên lowercase:
  - `admin`
  - `owner`
- nếu framework auth cần string compare, không trộn `Admin` và `admin`

## 16. Không làm

- không để query đi qua domain nếu không cần
- không để command thao tác thẳng model bỏ qua domain
- không kéo `domain error message` trả thẳng ra application response
- không để handler tự đọc lung tung nhiều bảng nếu đã có abstraction phù hợp
- không để service khác tự đọc DB của service source-of-truth
- không để migration mỗi service một chỗ
- không đặt `DbContextFactory` ở folder khác với `DbContext`
- không đặt migration trong `Persistence` chỉ vì model nằm ở đó

## 17. Quick review checklist

Khi review một PR backend service, check nhanh:

1. query có đi `model -> response` không
2. command có đi qua domain không
3. response có nằm đúng `Responses/` không
4. mapper có nằm đúng `Mappers/` không
5. validator có được tách khỏi handler không
6. handler có đang làm quá nhiều việc không
7. route/controller có đúng convention chung không
8. cross-service access có đi qua abstraction/integration không
9. nếu có EF thay đổi: `DbContextFactory`, folder migration, namespace migration, snapshot có đúng convention không

## 18. Operational note

Nếu cần tạo hoặc apply migration trong workspace hiện tại, ưu tiên dùng script chung:

- `Workspace/scripts/add-ef-migration.ps1`
- `Workspace/scripts/update-ef-database.ps1`

Lý do:

- script đã validate `DbContextFactory`
- script build service ở chế độ single-thread
- script giảm risk fail ảo do MSBuild song song trong workspace hiện tại
```

---

## Phụ lục: Quick Structure Map

Phần này chỉ là chú thích bổ sung để đọc nhanh hơn.

Không thay thế nội dung skill ở trên.

### Sơ đồ nhớ nhanh

```text
<Service>.API
├─ Application      -> Use case
├─ Domain           -> Business core
├─ Infrastructure   -> DB + external systems
├─ Presentation     -> HTTP/gRPC entrypoint
├─ Consumers        -> Async message entrypoint
└─ Program.cs       -> Bootstrap
```

### Cây thư mục chuẩn

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

### Mapping nhanh theo loại file

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

### Ghi chú về `Consumers`

- ưu tiên `Consumers/` ở root service
- không ưu tiên `Infrastructure/Consumers`
- xem `Consumers` là một entrypoint của service, ngang hàng với `Presentation`
- không kéo `Grpc/` ra root service
- `gRPC` vẫn thuộc `Presentation/Grpc` vì nó là synchronous interface
