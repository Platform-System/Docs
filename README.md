# 🚀 Platform System 

Bản đồ tri thức hệ thống Microservices Polyrepo.

---

## 🗺️ Master Map

### 1. User Interfaces (Hệ thống giao diện)
- [Platform.PortalUI](https://github.com/Platform-System/Platform.PortalUI): Cổng mua sắm trực tuyến Nexus dành cho khách hàng.
- [Platform.MerchantUI](https://github.com/Platform-System/Platform.MerchantUI): Cổng thông tin và quản lý dành cho Đối tác/Người bán (Merchant).
- [Platform.AdminUI](https://github.com/Platform-System/Platform.AdminUI): Hệ thống quản trị tập trung dành cho Ban điều hành (Admin).

### 2. Entry Point (Gateway)
- [Platform.Gateway](https://github.com/Platform-System/Platform.Gateway): Cổng tiếp nhận & Điều hướng yêu cầu (Gateway).

### 3. APIs (Services nghiệp vụ)
- [Platform.Catalog.API](https://github.com/Platform-System/Platform.Catalog.API): Quản lý Sản phẩm & Danh mục.
- [Platform.Identity.API](https://github.com/Platform-System/Platform.Identity.API): Quản lý Tài khoản & Xác thực người dùng.
- [Platform.Ordering.API](https://github.com/Platform-System/Platform.Ordering.API): Quản lý Đặt hàng & Đơn hàng.
- [Platform.Store.API](https://github.com/Platform-System/Platform.Store.API): Quản lý Cửa hàng & Sản phẩm bán lẻ.
- [Platform.Payment.API](https://github.com/Platform-System/Platform.Payment.API): Quản lý Thanh toán & Giao dịch.

### 4. Functions (Serverless Modules)
- [Platform.ProductCoverUpload.Function](https://github.com/Platform-System/Platform.ProductCoverUpload.Function): Serverless function xử lý tải lên ảnh bìa sản phẩm.
- [Platform.StoreImageUpload.Function](https://github.com/Platform-System/Platform.StoreImageUpload.Function): Serverless function xử lý tải lên ảnh cửa hàng.

### 5. Core (Xương sống kiến trúc)
- [Platform.Api](https://github.com/Platform-System/Platform.Api): Thư viện cơ sở cho các Web APIs.
- [Platform.Application](https://github.com/Platform-System/Platform.Application): Quy trình xử lý hồ sơ (Use Cases, MediatR).
- [Platform.Contracts](https://github.com/Platform-System/Platform.Contracts): Ngôn ngữ liên lạc chung (Integration Events).
- [Platform.Domain](https://github.com/Platform-System/Platform.Domain): Logic nghiệp vụ cốt lõi (Entities, Rules).
- [Platform.Infrastructure](https://github.com/Platform-System/Platform.Infrastructure): Hiện thực hóa công nghệ (RabbitMQ, Postgres).

### 6. Foundation (Hạt nhân dùng chung)
- [Platform.BuildingBlocks](https://github.com/Platform-System/Platform.BuildingBlocks): Các công cụ lập trình cơ bản.
- [Platform.SharedKernel](https://github.com/Platform-System/Platform.SharedKernel): Hạt nhân nghiệp vụ dùng chung.
- [Platform.SystemContext](https://github.com/Platform-System/Platform.SystemContext): Ngữ cảnh người dùng & Hệ thống.

### 7. Technical (Tiện ích công nghệ)
- [Platform.Email](https://github.com/Platform-System/Platform.Email): Dịch vụ hỗ trợ gửi thông báo Email.
- [Platform.Messaging](https://github.com/Platform-System/Platform.Messaging): Cơ chế giao tiếp Event-Driven (RabbitMQ).

### 8. DevOps Automation (CI-CD & IaC)
- [Platform.CI-CD](https://github.com/Platform-System/Platform.CI-CD): Quy trình tự động hóa GitHub Actions.
- [Platform.IaC](https://github.com/Platform-System/Platform.IaC): Cấu hình hạ tầng Docker.

### 9. System Documentation (Docs & Profile)
- [Platform.Docs](https://github.com/Platform-System/Platform.Docs): Tài liệu hệ thống Master.
- [Platform.Profile](https://github.com/Platform-System/.github): Hồ sơ hệ thống và Dashboards.

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
- **CI/CD Tập trung**: Mọi quy trình CI/CD BẮT BUỘC phải sử dụng template chuẩn tại repo `Platform.CI-CD`. KHÔNG ĐƯỢC tự tạo quy trình CI/CD rời rạc trong từng project. Nếu template thiếu tính năng, phải cập nhật trực tiếp vào file template trong `Platform.CI-CD`.
- **Tái sử dụng Code**: Chủ động tìm kiếm và đọc các repository xung quanh (ví dụ: `BuildingBlocks`, `Common`, v.v.) để tái sử dụng code, pattern hoặc thư viện đã có. KHÔNG tự code lại từ đầu nếu thành phần đó đã tồn tại trong hệ sinh thái.
- **Chiến lược Xác thực (JWT)**: Toàn bộ downstream HTTP APIs có user context phải đi theo hướng `every service validates JWT`. `Platform.Gateway` có thể xác thực trước ở edge, nhưng KHÔNG phải trust boundary duy nhất.
- **Nguồn danh tính chuẩn**: Các downstream service BẮT BUỘC phải lấy user identity từ `HttpContext.User.Claims` sau khi JWT đã được xác thực trong chính service đó. KHÔNG được dùng `X-User-*` headers làm nguồn danh tính chính cho business logic hoặc authorization.
- **Claims chuẩn toàn hệ**: Khi cần đọc user context, ưu tiên thống nhất theo các claim `sub`, `email`, `preferred_username`.
- **Chuẩn triển khai cho HTTP APIs**: Với các APIs nhận request từ client hoặc có user context, AI phải ưu tiên mẫu triển khai gồm `AddKeycloakAuthentication(builder.Configuration)`, `AddAuthorization()`, `UseAuthentication()`, `UseAuthorization()`, và gắn `[Authorize]` cho controller/action cần bảo vệ.
