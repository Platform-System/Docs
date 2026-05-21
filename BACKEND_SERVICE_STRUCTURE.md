# Backend Service Structure

Tai lieu nay chot "ban do src" cho cac backend service trong he `Platform.*.API`.

Muc tieu:

- nhin folder la biet file nen nam o dau
- tao feature moi ma khong bi dat sai tang
- giu style dong nhat giua cac service

## Cây thu muc chuan

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

## Y nghia tung tang

- `Application`
  - chua use case
  - noi dat `Command`, `Query`, `Handler`, `Validator`, `Response`, `Mapper`
  - neu muon biet service "lam gi" thi tim o day

- `Domain`
  - chua business core
  - noi dat `Entity`, `Enum`, `Error`, `ValueObject`, `Domain Event`
  - khong phu thuoc HTTP, DB, gRPC

- `Infrastructure`
  - chua DB va ket noi ben ngoai
  - noi dat `DbContext`, migration, EF model/configuration, options, gRPC/HTTP client, provider, outbox

- `Presentation`
  - chua HTTP controller va gRPC service
  - nen mong
  - nhan request, goi command/query, tra response

- `Consumers`
  - chua message consumer
  - xem nhu mot entrypoint cua service, ngang hang voi `Presentation`
  - convention chot la dat `Consumers/` o root service, khong uu tien `Infrastructure/Consumers`

- `Program.cs`
  - entry point bootstrap service
  - chi nen chua wiring nhu:
    - `AddApplication(...)`
    - `Add<Service>Infrastructure(...)`
    - auth, runtime, swagger
    - `ApplyMigrationsAsync<TDbContext>()`
    - `MapControllers()`, `MapGrpcService(...)`

## Dat file vao dau

- them API moi
  - `Presentation/Http`
  - va logic o `Application/Features/<FeatureName>`

- them gRPC endpoint moi
  - `Presentation/Grpc`

- them business rule, entity, error nghiep vu
  - `Domain`

- them `DbContext`, migration, EF model, EF configuration
  - `Infrastructure/Data`
  - `Infrastructure/Persistence/Models`
  - `Infrastructure/Persistence/Configurations`

- them config options
  - `Infrastructure/Configurations`

- them client goi service khac
  - `Infrastructure/Integrations`

- them payment provider / sandbox provider / strategy
  - `Infrastructure/Providers`

- them outbox dispatcher / outbox writer
  - `Infrastructure/Outbox`

- them consumer RabbitMQ / MassTransit
  - `Consumers`

## Rule nho nhanh

- `Presentation` = cua vao
- `Application` = xu ly use case
- `Domain` = luat nghiep vu cot loi
- `Infrastructure` = cham DB va ben ngoai
- `Consumers` = cua vao async

## Notes

- Khac biet do domain la binh thuong.
  - vi du service co `Providers` hoac `Outbox` khong co nghia la lech style
- Lech style chi xay ra khi dat sai tang.
  - vi du business logic day trong controller
  - hoac migration nam sai cho
  - hoac consumer bi chon vao folder khong ro vai tro
