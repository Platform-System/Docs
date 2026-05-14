# Service Boundaries And API Conventions

Tai lieu nay gop 2 muc tieu vao cung 1 cho:

- boundary va rule kien truc giua cac service
- convention code/style chung cho cac API service trong repo

Pham vi ap dung:

- `Platform.Catalog.API`
- `Platform.Store.API`
- `Platform.Ordering.API`
- `Platform.Identity.API`

## 1. Rule cot loi

- `query -> model -> response`
- `command -> domain -> response`
- handler chi orchestration
- handler khong tu keo rule domain/error message len application response
- neu co rule cheo subdomain/service:
  - tao abstraction truoc
  - roi moi doi implementation local -> gRPC

## 2. Service boundary rule

- moi service la source-of-truth cho data va business rule cua no
- service khac khong query truc tiep persistence cua service do
- cross-service access phai di qua abstraction hoac integration endpoint ro rang

Hien tai boundary mong muon:

- `Catalog` giu product, category, stock
- `Store` giu store profile, membership, verification, store policy
- `Ordering` giu cart, order, payment snapshot phuc vu checkout va history
- `Identity` giu user profile noi bo va identity-related integration surface

## 3. Query rule

- query doc tu `model`
- query map thang `model -> response`
- query khong di vong qua domain neu chi la read-only data
- neu query can doc rule/du lieu cheo feature nhieu lan, uu tien read abstraction/service helper

## 4. Command rule

- command di qua `domain`
- validate business rule o domain hoac policy service
- validate format/basic input o validator
- xong moi map ve persistence/response
- khong de command thao tac thang model bo qua domain

## 5. Mapper rule

- `PersistenceMapper`: map `domain <-> model`
- `ResponseMapper`: map `domain -> response` hoac `model -> response` voi query
- khong map tay dai trong handler neu da co mapper

## 6. Handler rule

- handler chi dieu phoi
- khong nhet business rule dai vao handler neu rule do thuoc domain/subdomain khac
- khong query truc tiep subdomain/service khac neu da co abstraction phu hop
- loi tra ve application nen uu tien `Result<T>.Failure(statusCode, message)`

## 7. Feature folder rule

Moi feature nen uu tien cau truc:

```text
Application/
  Features/<FeatureName>/
    Commands/
    Queries/
    Mappers/
    Responses/
    Services/
```

Y nghia:

- `Commands` chua `Command`, `Handler`, `Request`, `Validator`
- `Queries` chua `Query`, `Handler`
- `Mappers` chua `PersistenceMapper`, `ResponseMapper`, mapper nho theo feature
- `Responses` chua DTO tra ra API/application
- `Services` chi dung khi feature can read/service helper noi bo ro rang

Khong nen:

- dat class cua feature nay trong folder cua feature khac
- giu `Shared` cho response moi neu co the dat vao `Responses`

## 8. Response DTO rule

- response DTO dung `sealed class`
- uu tien `init` cho field read-only response
- dat trong folder `Responses`
- ten ket thuc bang `Response`

Vi du:

- `StoreDetailsResponse`
- `UserResponse`
- `CurrentUserResponse`

Pattern cu can han che mo rong:

- `Features/<Feature>/Shared/<Something>Response.cs`

## 9. Validation rule

- command co input request thi uu tien co `AbstractValidator<TCommand>`
- validation format/basic rule nam o validator
- business rule nam o domain hoac policy service
- khong validate input thu cong trong handler neu da co validator

## 10. Controller and route rule

- controller dung ten so nhieu khi dai dien resource:
  - `UsersController`
  - `ProductsController`
  - `StoresController`
  - `OrdersController`
- route uu tien explicit thay vi phu thuoc `[controller]` neu dang chot API public lau dai
- tach ro public surface va manage surface neu khac audience

Vi du:

- public: `api/stores`
- manage: `api/manage/stores`

## 11. Integration rule

- service source-of-truth cung cap abstraction hoac integration endpoint ro rang
- gRPC/controller integration nen map qua helper rieng:
  - `...IntegrationMapper`
  - `...IntegrationResponses`
- service goi sang service khac thi dung abstraction de de doi local implementation sang gRPC client

## 12. Naming rule

- role name dung mot chuan duy nhat trong toan repo
- de xuat chot lowercase:
  - `admin`
  - `owner`
- neu framework auth can string compare, khong tron `Admin` va `admin`

## 13. Khong lam

- khong de query di qua domain neu khong can
- khong de command thao tac thang model bo qua domain
- khong keo `domain error message` tra thang ra application response
- khong de handler tu doc lung tung nhieu bang neu da co abstraction phu hop
- khong de service khac tu doc DB cua service source-of-truth

## 14. Service alignment snapshot

### `Platform.Store.API`

Trang thai:

- dang gan convention nhat
- co `Responses`, `Mappers`, `Services`
- co tach public va manage route kha ro

Con lech:

- naming controller van tron giua resource controller va action-oriented controller

Nen chinh tiep:

1. review xem `StoreManagementController` va `StoreVerificationController` co can doi ten theo bounded surface convention hay giu nguyen

### `Platform.Identity.API`

Trang thai:

- da duoc keo gan convention moi
- co `Responses`, `Mappers`, `Services`
- da co `IUserReadService`
- controller da doi sang `UsersController`

Con lech:

- folder `Shared` rong neu IDE dang giu handle thi xoa sau

Nen chinh tiep:

1. xoa folder `Shared` rong

### `Platform.Catalog.API`

Trang thai:

- layer va abstraction tuong doi on
- da co `IStoreReadService`, `IStorePolicyService`
- co mapper ro cho nhieu feature
- response DTO da vao `Responses`
- route explicit da ro hon cho `products`, `categories`, `product-medias`

Con lech:

- mot so policy/approval flow naming con dai va khong deu tay
- mot so surface lien quan den store/product ownership van tach o `StoreProductsController`

Nen chinh tiep:

1. giu `Shared` chi cho utility khong phai response neu thuc su can
2. can nhac co nen gom lai surface `StoreProductsController` vao bounded route scheme ro hon nua hay khong

### `Platform.Ordering.API`

Trang thai:

- feature split ro
- validation command kha day du
- da co abstraction integration `ICatalogClient`
- response DTO da vao `Responses`
- route da doi sang explicit resource-style cho `carts` va `orders`

Con lech:

- folder `Shared` rong neu IDE dang giu handle thi xoa sau
- command/request naming ben trong van mang dau vet action cu, du route ben ngoai da sach hon

Nen chinh tiep:

1. xoa folder `Shared` rong
2. can nhac doi ten command/request noi bo neu muon dong bo hon voi route moi

## 15. Priority refactor order

Nen lam theo thu tu:

1. xoa folder `Shared` rong va namespace cu sau khi IDE nha handle
2. chot route explicit/public cho `Catalog`
3. review naming command/request noi bo cua `Ordering` neu muon sat route moi hon
4. review naming controller manage surface cua `Store`

## 16. Quick review checklist

Khi review mot PR trong service API, check nhanh:

1. query co di `model -> response` khong
2. command co di qua domain khong
3. response co nam dung `Responses/` khong
4. mapper co nam dung `Mappers/` khong
5. validator co duoc tach khoi handler khong
6. handler co dang lam qua nhieu viec khong
7. route/controller co dung convention chung khong
8. role string co dung format da chot khong
9. cross-service access co di qua abstraction/integration khong
