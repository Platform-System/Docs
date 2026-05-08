# 🚀 Platform System 

Bản đồ tri thức hệ thống Microservices Polyrepo.

---

## 🗺️ Master Map

### 1. UI (Giao diện người dùng)
- [<img src="./assets/labels/ui.svg" alt="Platform.UI">](https://github.com/Platform-System/Platform.UI): Giao diện cổng thông tin Nexus hiện đại.

### 2. Entry Point (Gateway)
- [<img src="./assets/labels/gateway.svg" alt="Platform.Gateway">](https://github.com/Platform-System/Platform.Gateway): Cổng tiếp nhận & Điều hướng yêu cầu (Gateway).

### 3. APIs (Services nghiệp vụ)
- [<img src="./assets/labels/catalog_service.svg" alt="Platform.Catalog.API">](https://github.com/Platform-System/Platform.Catalog.API): Quản lý Sản phẩm & Danh mục.
- [<img src="./assets/labels/identity_service.svg" alt="Platform.Identity.API">](https://github.com/Platform-System/Platform.Identity.API): Quản lý Tài khoản & Xác thực người dùng.
- [<img src="./assets/labels/ordering_service.svg" alt="Platform.Ordering.API">](https://github.com/Platform-System/Platform.Ordering.API): Quản lý Đặt hàng & Đơn hàng.
- [<img src="./assets/labels/store_service.svg" alt="Platform.Store.API">](https://github.com/Platform-System/Platform.Store.API): Quản lý Cửa hàng & Sản phẩm bán lẻ.

### 4. Functions (Serverless Modules)
- [<img src="./assets/labels/product_cover_upload.svg" alt="Platform.ProductCoverUpload.Function">](https://github.com/Platform-System/Platform.ProductCoverUpload.Function): Serverless function xử lý tải lên ảnh bìa sản phẩm.

### 5. Core (Xương sống kiến trúc)
- [<img src="./assets/labels/api.svg" alt="Platform.Api">](https://github.com/Platform-System/Platform.Api): Thư viện cơ sở cho các Web APIs.
- [<img src="./assets/labels/application.svg" alt="Platform.Application">](https://github.com/Platform-System/Platform.Application): Quy trình xử lý hồ sơ (Use Cases, MediatR).
- [<img src="./assets/labels/contracts.svg" alt="Platform.Contracts">](https://github.com/Platform-System/Platform.Contracts): Ngôn ngữ liên lạc chung (Integration Events).
- [<img src="./assets/labels/domain.svg" alt="Platform.Domain">](https://github.com/Platform-System/Platform.Domain): Logic nghiệp vụ cốt lõi (Entities, Rules).
- [<img src="./assets/labels/infrastructure.svg" alt="Platform.Infrastructure">](https://github.com/Platform-System/Platform.Infrastructure): Hiện thực hóa công nghệ (RabbitMQ, Postgres).

### 6. Foundation (Hạt nhân dùng chung)
- [<img src="./assets/labels/buildingblocks.svg" alt="Platform.BuildingBlocks">](https://github.com/Platform-System/Platform.BuildingBlocks): Các công cụ lập trình cơ bản.
- [<img src="./assets/labels/shared_kernel.svg" alt="Platform.SharedKernel">](https://github.com/Platform-System/Platform.SharedKernel): Hạt nhân nghiệp vụ dùng chung.
- [<img src="./assets/labels/systemcontext.svg" alt="Platform.SystemContext">](https://github.com/Platform-System/Platform.SystemContext): Ngữ cảnh người dùng & Hệ thống.

### 7. Technical (Tiện ích công nghệ)
- [<img src="./assets/labels/email_service.svg" alt="Platform.Email">](https://github.com/Platform-System/Platform.Email): Dịch vụ hỗ trợ gửi thông báo Email.
- [<img src="./assets/labels/messaging.svg" alt="Platform.Messaging">](https://github.com/Platform-System/Platform.Messaging): Cơ chế giao tiếp Event-Driven (RabbitMQ).

### 8. Ops (Vận hành)
- [<img src="./assets/labels/ci_cd.svg" alt="Platform.CI-CD">](https://github.com/Platform-System/Platform.CI-CD): Quy trình tự động hóa GitHub Actions.
- [<img src="./assets/labels/docs.svg" alt="Platform.Docs">](https://github.com/Platform-System/Platform.Docs): Tài liệu hệ thống Master.
- [<img src="./assets/labels/iac.svg" alt="Platform.IaC">](https://github.com/Platform-System/Platform.IaC): Cấu hình hạ tầng Docker.
- [<img src="./assets/labels/profile_label.svg" alt="Platform.Profile">](https://github.com/Platform-System/.github): Hồ sơ hệ thống và Dashboards.




---
## 🤖 AI Context & Instruction (Dành cho AI)
Nếu bạn là AI hỗ trợ dự án này, hãy tuân thủ:
- **Ngữ cảnh**: Hệ thống Microservices Polyrepo (20 repositories).
- **Kiến trúc**: Clean Architecture, DDD, Event-Driven.
- **Nghiệm vụ**: Review code, tối ưu hóa logic nghiệp vụ và đảm bảo tính đồng nhất giữa các repository.
- **Tiêu chuẩn**: Luôn kiểm tra các `Contracts` trước khi đề xuất thay đổi Integration Events.
- **GIỚI HẠN QUYỀN (QUAN TRỌNG)**: User đang học Microservices, nên AI tuyệt đối KHÔNG ĐƯỢC dùng tool can thiệp sửa code trực tiếp. AI chỉ được đưa ra code mẫu trong khung chat để user tự gõ lại và rèn luyện. BẮT BUỘC phải có lệnh cho phép trước khi thao tác push/commit đối với toàn bộ các dự án Code. **(☀️ NGOẠI LỆ: AI ĐƯỢC PHÉP ĐẶC CÁCH tự động chỉnh sửa và push thẳng lên riêng Repo `Platform.Docs` này mà không cần xin phép)**.
- **Quy chuẩn Commit**: Mọi mã commit đẩy lên đều phải tuân theo chuẩn Conventional format (feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert) và BẮT BUỘC viết bằng Tiếng Anh (English ONLY).
- **Cleanup Code**: Trước khi tiến hành Commit/Push (nếu được phép), AI BẮT BUỘC phải soi lại toàn bộ C# class và dọn dẹp xóa sạch các đoạn khai báo namespace (`using`) bị thừa thãi. Đảm bảo source code luôn Clean trước khi lên Remote.
- **CI/CD Tập trung**: Mọi quy trình CI/CD BẮT BUỘC phải sử dụng template chuẩn tại repo `Platform.CI-CD`. KHÔNG ĐƯỢC tự tạo quy trình CI/CD rời rạc trong từng project. Nếu template thiếu tính năng, phải cập nhật trực tiếp vào file template trong `Platform.CI-CD`.
- **Tái sử dụng Code**: Chủ động tìm kiếm và đọc các repository xung quanh (ví dụ: `BuildingBlocks`, `Common`, v.v.) để tái sử dụng code, pattern hoặc thư viện đã có. KHÔNG tự code lại từ đầu nếu thành phần đó đã tồn tại trong hệ sinh thái.
- **Chiến lược Xác thực (JWT)**: Toàn bộ downstream HTTP APIs có user context phải đi theo hướng `every service validates JWT`. `Platform.Gateway` có thể xác thực trước ở edge, nhưng KHÔNG phải trust boundary duy nhất.
- **Nguồn danh tính chuẩn**: Các downstream service BẮT BUỘC phải lấy user identity từ `HttpContext.User.Claims` sau khi JWT đã được xác thực trong chính service đó. KHÔNG được dùng `X-User-*` headers làm nguồn danh tính chính cho business logic hoặc authorization.
- **Claims chuẩn toàn hệ**: Khi cần đọc user context, ưu tiên thống nhất theo các claim `sub`, `email`, `preferred_username`.
- **Chuẩn triển khai cho HTTP APIs**: Với các APIs nhận request từ client hoặc có user context, AI phải ưu tiên mẫu triển khai gồm `AddKeycloakAuthentication(builder.Configuration)`, `AddAuthorization()`, `UseAuthentication()`, `UseAuthorization()`, và gắn `[Authorize]` cho controller/action cần bảo vệ.
