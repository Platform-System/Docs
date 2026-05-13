# Store Business Rules

- Hệ thống hiện tại là `store-centric`: mọi quyền tạo, quản lý và publish sản phẩm đều đi qua membership của user trong store.
- Một user hiện chỉ thuộc về một store tại một thời điểm.
- Mỗi store có `Slug` duy nhất và một bộ policy duy nhất gồm shipping, return, warranty.
- Khi tạo store, store bắt đầu ở `Draft`, chưa verified, và người tạo trở thành `Owner`.
- Owner có thể request verification; admin approve sẽ chuyển store sang `Active` và `IsVerified = true`.
- Chỉ owner của verified store mới được mời thêm member, và chỉ mời được `Manager` hoặc `Staff`.
- Owner luôn có quyền publish trực tiếp; owner có thể cấp hoặc thu hồi quyền publish trực tiếp cho member active.
- User chỉ tạo được sản phẩm khi thuộc một store hợp lệ; nếu store chưa verified thì chỉ owner mới được tạo.
- Product mới luôn bắt đầu ở `Draft`.
- Nếu store chưa verified: owner submit product sẽ vào `PendingAdminReview`; manager/staff không được submit.
- Nếu store đã verified: owner hoặc member có quyền publish trực tiếp có thể đưa product lên `Active`; member còn lại sẽ vào `PendingOwnerReview`.
- Product ở `PendingOwnerReview` chỉ owner approve được; product ở `PendingAdminReview` chỉ admin approve được.
- Chỉ creator của product hoặc owner của store mới được update/delete product; product đã `Active` hiện không update bằng flow hiện tại.
- Public listing chỉ hiển thị store `Active` và product `Active`.
- Store detail theo slug hiện chỉ chặn store `Deleted`, chưa chặn `Draft`, `PendingVerification`, `Suspended`.
- Danh sách product theo `store slug` hiện vẫn yêu cầu user đã authenticate.
- Owner có màn hình/list riêng cho product `PendingOwnerReview` của store mình.
- User có list riêng cho các product tự tạo đang ở `Draft`, `PendingOwnerReview`, `PendingAdminReview`.
- Admin có list riêng cho các product đang `PendingAdminReview`.

## Summary

`store` là trust boundary của business, `owner` là người quyết định trong store, còn `admin` là lớp duyệt cuối cho các trường hợp store chưa đủ điều kiện tự publish.
