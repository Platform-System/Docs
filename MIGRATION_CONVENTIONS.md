# Migration Conventions For Backend Services

Tai lieu nay chot convention cho `DbContext`, `DbContextFactory` va `EF Core Migrations` trong he backend.

Pham vi ap dung:

- `Platform.Catalog.API`
- `Platform.Identity.API`
- `Platform.Ordering.API`
- `Platform.Payment.API`
- `Platform.Store.API`
- `Platform.Wallet.API`

## 1. Muc tieu

- giam tinh trang moi service dat migration mot kieu
- de `dotnet ef` chay on dinh va de doan
- de nguoi moi vao repo nhin phat biet ngay file nao thuoc persistence bootstrap
- giam risk namespace lech, migration snapshot lech, hoac add migration sai folder

## 2. Convention chot

Moi service co database rieng se theo cau truc:

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

Y nghia:

- `Data/` la noi chua thanh phan bootstrap EF:
  - `DbContext`
  - `IDesignTimeDbContextFactory`
  - `Migrations`
- `Persistence/` la noi chua persistence shape:
  - `Models`
  - `EntityTypeConfiguration`
  - persistence helper khac neu co

## 3. Namespace convention

- `DbContext`: `Platform.<Service>.API.Infrastructure.Data`
- `DbContextFactory`: `Platform.<Service>.API.Infrastructure.Data`
- `Migrations`: `Platform.<Service>.API.Infrastructure.Data.Migrations`
- `Models`: `Platform.<Service>.API.Infrastructure.Persistence.Models`
- `Configurations`: `Platform.<Service>.API.Infrastructure.Persistence.Configurations`

Khong tron:

- `Infrastructure.Data.Migrations`
- `Infrastructure.Persistence.Migrations`

Trong cung mot service chi duoc co mot chuan duy nhat:

- `Infrastructure.Data.Migrations`

## 4. Hien trang da chot trong repo

Sau khi cleanup:

- `Catalog`: dung `Infrastructure/Data/Migrations`
- `Identity`: dung `Infrastructure/Data/Migrations`
- `Store`: dung `Infrastructure/Data/Migrations`
- `Payment`: da dua ve `Infrastructure/Data/Migrations`
- `Wallet`: da dua ve `Infrastructure/Data/Migrations`
- `Ordering`: da co `OrderingDbContextFactory`, chua co migration nhung khi tao moi phai theo `Infrastructure/Data/Migrations`

## 5. Rule cho DbContext

- moi service co database rieng phai co dung 1 `DbContext` chinh
- `DbContext` dat trong `Infrastructure/Data`
- `DbContext` map toi persistence model trong `Infrastructure/Persistence/Models`
- `OnModelCreating` apply configuration tu assembly hien tai

Pattern mong muon:

```csharp
public sealed class ServiceDbContext : BaseDbContext
{
    public ServiceDbContext(DbContextOptions<ServiceDbContext> options, ICurrentUserProvider? currentUserProvider = null)
        : base(options, currentUserProvider)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(ServiceDbContext).Assembly);
        base.OnModelCreating(modelBuilder);
    }
}
```

## 6. Rule cho DbContextFactory

- moi service co `DbContext` phai co `IDesignTimeDbContextFactory<TContext>`
- factory dat cung folder voi `DbContext`: `Infrastructure/Data`
- factory doc `appsettings.json` va `appsettings.Development.json`
- factory dung dung connection string cua service do

Loi ich:

- `dotnet ef migrations add` khong phu thuoc host bootstrapping
- giam tinh trang service nay add migration duoc, service kia fail vi khong tao duoc `DbContext`

## 7. Rule cho migration files

- migration phai nam duoi `Infrastructure/Data/Migrations`
- khong dat migration trong `Persistence`
- namespace migration phai theo folder that su
- snapshot phai di cung migration
- khong edit tay `Designer.cs` neu khong that su can rescue conflict

Dat ten migration:

- dung ten mo ta business/persistence change ro rang
- uu tien:
  - `InitialCreate`
  - `AddPaymentOutbox`
  - `AddStoreMedia`
  - `RenameOrderStatusIndex`

Khong nen dat:

- `UpdateDb`
- `Fix`
- `TestMigration`

## 8. Cac lenh nen dung

Vi du voi `Ordering`:

```powershell
dotnet ef migrations add InitialCreate `
  --project Platform.Ordering.API/Platform.Ordering.API.csproj `
  --startup-project Platform.Ordering.API/Platform.Ordering.API.csproj `
  --output-dir Infrastructure/Data/Migrations
```

Update database:

```powershell
dotnet ef database update `
  --project Platform.Ordering.API/Platform.Ordering.API.csproj `
  --startup-project Platform.Ordering.API/Platform.Ordering.API.csproj
```

Neu dang tao migration cho service khac, giu nguyen pattern:

- `--project` = service project chua `DbContext`
- `--startup-project` = chinh service do
- `--output-dir` = `Infrastructure/Data/Migrations`

## 9. Checklist khi tao service moi co database

1. tao `Infrastructure/Data/<Service>DbContext.cs`
2. tao `Infrastructure/Data/<Service>DbContextFactory.cs`
3. tao `Infrastructure/Persistence/Models`
4. tao `Infrastructure/Persistence/Configurations`
5. dang ky `AddDbContext` trong `Infrastructure/DependencyInjection`
6. tao migration dau tien vao `Infrastructure/Data/Migrations`
7. verify namespace migration dung `Infrastructure.Data.Migrations`
8. chot connection string key rieng cho service

## 10. Checklist khi review PR co migration

1. migration co nam dung `Infrastructure/Data/Migrations` khong
2. namespace migration co dung `Infrastructure.Data.Migrations` khong
3. service da co `DbContextFactory` chua
4. model/configuration co van nam o `Persistence` khong
5. migration name co mo ta du thay doi khong
6. snapshot co duoc update cung migration khong
7. co dau hieu generate nham startup project hoac nham context khong

## 11. Khong lam

- khong de migration moi service mot cho
- khong dat `DbContextFactory` o folder khac voi `DbContext`
- khong dat migration trong `Persistence` chi vi model nam o do
- khong tron nhieu namespace migration trong cung service
- khong commit migration moi ma quen snapshot

## 12. Huong xu ly cho Ordering

`Platform.Ordering.API` hien co:

- `OrderingDbContext`
- `Persistence/Models`
- `Persistence/Configurations`
- `OrderingDbContextFactory`

Con thieu:

- migration dau tien
- snapshot dau tien

Khi bat dau migration cho `Ordering`, phai tao theo dung command o muc 8 va khong duoc dat sang `Infrastructure/Persistence/Migrations`.

## 13. Ket luan

Chuan repo cho EF Core migration backend la:

- `DbContext` va `DbContextFactory` o `Infrastructure/Data`
- `Migrations` o `Infrastructure/Data/Migrations`
- `Models` va `Configurations` o `Infrastructure/Persistence`

Neu mot PR moi khong theo 3 diem nay thi nen xem la lech convention va can sua truoc khi merge.
