# 🚀 System Workspace

Bản đồ tri thức hệ thống Microservices Polyrepo.

---

## 🗺️ Master Map

### 1. User Interfaces (Hệ thống giao diện)
- [PortalUI](https://github.com/System/PortalUI): Cổng mua sắm trực tuyến Nexus dành cho khách hàng.
- [MerchantUI](https://github.com/System/MerchantUI): Cổng thông tin và quản lý dành cho Đối tác/Người bán (Merchant).
- [AdminUI](https://github.com/System/AdminUI): Hệ thống quản trị tập trung dành cho Ban điều hành (Admin).

### 2. Entry Point (Gateway)
- [Gateway](https://github.com/System/Gateway): Cổng tiếp nhận & Điều hướng yêu cầu (Gateway).

### 3. APIs (Services nghiệp vụ)
- [Catalog.API](https://github.com/System/Catalog.API): Quản lý Sản phẩm & Danh mục.
- [Identity.API](https://github.com/System/Identity.API): Quản lý Tài khoản & Xác thực người dùng.
- [Ordering.API](https://github.com/System/Ordering.API): Quản lý Đặt hàng & Đơn hàng.
- [Store.API](https://github.com/System/Store.API): Quản lý Cửa hàng & Sản phẩm bán lẻ.
- [Payment.API](https://github.com/System/Payment.API): Quản lý Thanh toán & Giao dịch.

### 4. Functions (Serverless Modules)
- [ProductCoverUpload.Function](https://github.com/System/ProductCoverUpload.Function): Serverless function xử lý tải lên ảnh bìa sản phẩm.
- [StoreMediaUpload.Function](https://github.com/System/StoreMediaUpload.Function): Serverless function xử lý tải lên ảnh cửa hàng.

### 5. Core (Xương sống kiến trúc)
- [Api](https://github.com/System/Api): Thư viện cơ sở cho các Web APIs.
- [Application](https://github.com/System/Application): Quy trình xử lý hồ sơ (Use Cases, MediatR).
- [Contracts](https://github.com/System/Contracts): Ngôn ngữ liên lạc chung (Integration Events).
- [Domain](https://github.com/System/Domain): Logic nghiệp vụ cốt lõi (Entities, Rules).
- [Infrastructure](https://github.com/System/Infrastructure): Hiện thực hóa công nghệ (RabbitMQ, Postgres).

### 6. Foundation (Hạt nhân dùng chung)
- [BuildingBlocks](https://github.com/System/BuildingBlocks): Các công cụ lập trình cơ bản.
- [SharedKernel](https://github.com/System/SharedKernel): Hạt nhân nghiệp vụ dùng chung.
- [SystemContext](https://github.com/System/SystemContext): Ngữ cảnh người dùng & Hệ thống.

### 7. Technical (Tiện ích công nghệ)
- [Email](https://github.com/System/Email): Dịch vụ hỗ trợ gửi thông báo Email.
- [Messaging](https://github.com/System/Messaging): Cơ chế giao tiếp Event-Driven (RabbitMQ).

### 8. DevOps Automation (CI-CD & IaC)
- [CI-CD](https://github.com/System/CI-CD): Quy trình tự động hóa GitHub Actions.
- [IaC](https://github.com/System/IaC): Cấu hình hạ tầng Docker.

### 9. System Documentation (Docs & Profile)
- [Docs](https://github.com/System/Docs): Tài liệu hệ thống Master.
- [Profile](https://github.com/System/.github): Hồ sơ hệ thống và Dashboards.

## Frontend Docs

- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md): Kiến trúc phân lớp 3 tầng, quy tắc import bảo vệ đóng gói và cơ chế cấu hình môi trường động ở runtime.

## Backend Docs

- [Backend Service Structure](./BACKEND_SERVICE_STRUCTURE.md): Bản đồ src chuẩn cho các backend service để nhìn folder là biết file nên nằm ở đâu.
- [Kafka Flow Conventions](./KAFKA_FLOW_CONVENTIONS.md): Convention triển khai flow Kafka moi, gom outbox, consumer, retry, DLT va checklist test.
- [Migration Conventions](./MIGRATION_CONVENTIONS.md): Convention cho `DbContext`, `DbContextFactory` va `EF Core Migrations` trong cac backend service.
- [Microservice Project Arch Skill Backup](./MICROSERVICE_PROJECT_ARCH_SKILL.md): Ban backup trong repo cho local Codex skill `microservice-project-arch`.




---
## 🤖 AI Context & Instruction (Dành cho AI)
Nếu bạn là AI hỗ trợ dự án này, hãy tuân thủ:
- **Ngữ cảnh**: Hệ thống Microservices Polyrepo (20 repositories).
- **Kiến trúc**: Clean Architecture, DDD, Event-Driven.
- **Nghiệm vụ**: Review code, tối ưu hóa logic nghiệp vụ và đảm bảo tính đồng nhất giữa các repository.
- **Tiêu chuẩn**: Luôn kiểm tra các `Contracts` trước khi đề xuất thay đổi Integration Events.
- **QUYỀN THAO TÁC**: AI được phép tự động sử dụng các tool để chỉnh sửa code trực tiếp trên toàn bộ các repo trong hệ thống. Tuy nhiên, AI BẮT BUỘC phải xin phép và được sự đồng ý của User trước khi thực hiện bất kỳ thao tác Commit hoặc Push code nào lên Remote repository.
- **Quy chuẩn Commit**: Mọi mã commit đẩy lên đều phải tuân theo chuẩn Conventional format (feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert) và BẮT BUỘC viết bằng Tiếng Anh (English ONLY).
- **Cleanup Code**: Trước khi tiến hành Commit/Push (nếu được phép), AI BẮT BUỘC phải soi lại toàn bộ C# class và dọn dẹp xóa sạch các đoạn khai báo namespace (`using`) bị thừa thãi. Đảm bảo source code luôn Clean trước khi lên Remote.
- **CI/CD Tập trung**: Mọi quy trình CI/CD BẮT BUỘC phải sử dụng template chuẩn tại repo `CI-CD`. KHÔNG ĐƯỢC tự tạo quy trình CI/CD rời rạc trong từng project. Nếu template thiếu tính năng, phải cập nhật trực tiếp vào file template trong `CI-CD`.
- **Tái sử dụng Code**: Chủ động tìm kiếm và đọc các repository xung quanh (ví dụ: `BuildingBlocks`, `Common`, v.v.) để tái sử dụng code, pattern hoặc thư viện đã có. KHÔNG tự code lại từ đầu nếu thành phần đó đã tồn tại trong hệ sinh thái.
- **Chiến lược Xác thực (JWT)**: Toàn bộ downstream HTTP APIs có user context phải đi theo hướng `every service validates JWT`. `Gateway` có thể xác thực trước ở edge, nhưng KHÔNG phải trust boundary duy nhất.
- **Nguồn danh tính chuẩn**: Các downstream service BẮT BUỘC phải lấy user identity từ `HttpContext.User.Claims` sau khi JWT đã được xác thực trong chính service đó. KHÔNG được dùng `X-User-*` headers làm nguồn danh tính chính cho business logic hoặc authorization.
- **Claims chuẩn toàn hệ**: Khi cần đọc user context, ưu tiên thống nhất theo các claim `sub`, `email`, `preferred_username`.
- **Chuẩn triển khai cho HTTP APIs**: Với các APIs nhận request từ client hoặc có user context, AI phải ưu tiên mẫu triển khai gồm `AddKeycloakAuthentication(builder.Configuration)`, `AddAuthorization()`, `UseAuthentication()`, `UseAuthorization()`, và gắn `[Authorize]` cho controller/action cần bảo vệ.
