---
title: "Tuần 3: Xây dựng Backend NestJS"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

## 1. Mục tiêu

- Xây dựng module xử lý ảnh trong Backend NestJS.
- Tích hợp AWS SDK for JavaScript với Amazon S3.
- Kiểm tra định dạng, dung lượng và quyền sở hữu ảnh.
- Xây dựng luồng upload trực tiếp bằng pre-signed URL.
- Xây dựng xác thực JWT và API quản lý người dùng, công thức, nguyên liệu, danh mục, lượt thích và bình luận.
- Chuẩn bị migration PostgreSQL, health endpoint và Dockerfile production.

## 2. Kế hoạch công việc

> **Thời gian tuần 3:** Thứ 2, 06/07/2026 – Thứ 6, 10/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Thiết kế module, controller, service và DTO cho ảnh | Có cấu trúc Backend rõ ràng |
| Thứ 3 | Tích hợp S3 client và API upload qua Backend | Upload ảnh lên đúng object key |
| Thứ 4 | Thêm validation loại file, kích thước và metadata | Chặn file không hợp lệ trước khi lưu |
| Thứ 5 | Tạo API cấp pre-signed URL và API xác nhận upload | Giảm tải dữ liệu đi qua Backend |
| Thứ 6 | Hoàn thiện auth, recipe, like/comment API, migration, health check và Dockerfile | Backend sẵn sàng tích hợp và deploy |

## 3. Nội dung thực hiện

### 3.1. Module ảnh trong NestJS

- Tách `ImageModule`, `ImageController` và `ImageService`.
- Dùng DTO cùng `ValidationPipe` để kiểm tra request.
- Sinh object key theo user, recipe và UUID để tránh trùng tên.
- Chỉ sử dụng bốn trạng thái: `pending`, `processing`, `completed` và `failed`.
- Xây `AuthModule`, `UsersModule`, `RecipesModule`, `IngredientsModule`, `CategoriesModule`, `LikesModule` và `CommentsModule`.
- Thêm JWT/cookie authentication, ownership check, pagination và tìm kiếm công thức.

**Swagger hiển thị tài liệu và các endpoint của FoodieRecipe API:**

![Tổng quan FoodieRecipe API trên Swagger](/images/1-Worklog/1.3-Week3/swagger-overview.png)

**Nhập thông tin để kiểm thử API đăng nhập trực tiếp trên Swagger:**

![Gửi thông tin đăng nhập qua Swagger](/images/1-Worklog/1.3-Week3/swagger-login-request.png)

**API đăng nhập xử lý thành công và trả về thông tin người dùng:**

![Kết quả kiểm thử đăng nhập thành công trên Swagger](/images/1-Worklog/1.3-Week3/swagger-login-success.png)

### 3.2. Tích hợp Amazon S3

- Cấu hình S3 client bằng Region và thông tin xác thực từ biến môi trường.
- Gửi đúng `Content-Type` và metadata khi tạo object.
- Chỉ cấp các action S3 cần thiết cho Backend.
- Chuẩn hóa lỗi `AccessDenied`, object không tồn tại và upload bị gián đoạn.

### 3.3. Pre-signed URL

Backend tạo URL upload có thời hạn và ràng buộc object key. Sau khi trình duyệt upload trực tiếp lên S3, Frontend gọi API xác nhận; Backend kiểm tra object trước khi chuyển sang bước phân tích AI.

Hoàn thiện migration PostgreSQL cho dữ liệu nghiệp vụ, lượt thích và bình luận; bổ sung endpoint `/health` cùng Dockerfile multi-stage. Chạy unit test, migration và container local để xác nhận Backend sẵn sàng triển khai lên EC2.

**Kiểm tra dữ liệu và các bảng PostgreSQL sau khi chạy Prisma migration:**

![Kết quả Prisma migration được kiểm tra bằng Prisma Studio](/images/1-Worklog/1.3-Week3/prisma-migration-result.png)

**Kiểm tra PostgreSQL container bằng lệnh `docker ps`:**

![PostgreSQL container hoạt động ở trạng thái healthy](/images/1-Worklog/1.3-Week3/docker-postgres-running.png)

**Kiểm tra endpoint gốc của NestJS bằng lệnh `curl http://localhost:3001/api`:**

![NestJS API trả về Hello World thành công](/images/1-Worklog/1.3-Week3/api-hello-world.png)

## 4. Kiến thức và kỹ năng đạt được

- Tổ chức module, dependency injection và validation trong NestJS.
- Sử dụng AWS SDK với S3 command và pre-signed URL.
- Thiết kế API upload an toàn và có trạng thái.
- Xử lý lỗi và dọn object khi quy trình không hoàn tất.
- Thiết kế API nghiệp vụ, migration và Docker image production cho NestJS.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| File giả mạo MIME type | Kiểm tra cả MIME type, phần mở rộng và chữ ký file |
| URL hết hạn giữa lúc upload | Cho phép yêu cầu URL mới nhưng giữ nguyên image record |
| Upload xong nhưng không gọi xác nhận | Dùng trạng thái `pending` và tác vụ dọn object quá hạn |

## 6. Kết quả đầu ra

- Module ảnh NestJS tích hợp Amazon S3.
- API upload qua Backend và luồng pre-signed URL.
- Validation, quản lý trạng thái và quy trình xóa ảnh.
- Auth, user, recipe, ingredient, category API, migration và Dockerfile production.

## 7. Kế hoạch tuần tiếp theo

Xây dựng đầy đủ giao diện tài khoản, công thức, lượt thích, bình luận, tìm kiếm và upload ảnh bằng Next.js; tích hợp với các API NestJS.
