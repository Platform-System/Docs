# Store Business Rules

- Hệ thống hiện tại là `store-centric`: mọi quyền tạo, quản lý và publish sản phẩm đều đi qua membership của user trong store.
- Một user hiện chỉ thuộc về một store tại một thời điểm.
- Mỗi store có `Slug` duy nhất và một bộ policy duy nhất gồm shipping, return, warranty.
- Khi tạo store, store bắt đầu ở `Draft` và người tạo trở thành `Owner`.
- Owner có thể gửi yêu cầu kích hoạt store; admin approve sẽ chuyển store sang `Active`.
- Chỉ owner của store `Active` mới được mời thêm member, và chỉ mời được `Manager` hoặc `Staff`.
- Owner luôn có quyền publish trực tiếp; owner có thể cấp hoặc thu hồi quyền publish trực tiếp cho member active.
- User chỉ tạo được sản phẩm khi thuộc một store `Active`.
- Product mới luôn bắt đầu ở `Draft`.
- Nếu store chưa `Active`: không ai được submit/publish product, không được mời member, không được cấp quyền publish.
- Nếu store đã `Active`: owner hoặc member có quyền publish trực tiếp có thể đưa product lên `Active`; member còn lại sẽ vào `PendingOwnerReview`.
- Product ở `PendingOwnerReview` chỉ owner approve được; admin không còn duyệt product.
- Chỉ creator của product hoặc owner của store mới được update/delete product; product đã `Active` hiện không update bằng flow hiện tại.
- Public listing chỉ hiển thị store `Active` và product `Active`.
- Store detail theo slug hiện chỉ chặn store `Deleted`, chưa chặn `Draft`, `PendingActive`, `Suspended`.
- Danh sách product theo `store slug` là public, không yêu cầu authenticate.
- Owner có màn hình/list riêng cho product `PendingOwnerReview` của store mình.
- User có list riêng cho các product tự tạo đang ở `Draft`, `PendingOwnerReview`.

## Summary

`store` là trust boundary của business, `owner` là người quyết định trong store, còn `admin` là lớp duyệt kích hoạt cuối để store bước vào trạng thái `Active`.
