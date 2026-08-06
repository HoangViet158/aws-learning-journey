---
title: "Tuần 3: Backend NestJS và Amazon S3"
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

## 2. Kế hoạch công việc

> **Thời gian tuần 3:** Thứ 2, 06/07/2026 – Thứ 6, 10/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Thiết kế module, controller, service và DTO cho ảnh | Có cấu trúc Backend rõ ràng |
| Thứ 3 | Tích hợp S3 client và API upload qua Backend | Upload ảnh lên đúng object key |
| Thứ 4 | Thêm validation loại file, kích thước và metadata | Chặn file không hợp lệ trước khi lưu |
| Thứ 5 | Tạo API cấp pre-signed URL và API xác nhận upload | Giảm tải dữ liệu đi qua Backend |
| Thứ 6 | Xây API xem, thay thế, xóa ảnh và xử lý lỗi | Hoàn chỉnh vòng đời ảnh |

## 3. Nội dung thực hiện

### 3.1. Module ảnh trong NestJS

- Tách `ImageModule`, `ImageController` và `ImageService`.
- Dùng DTO cùng `ValidationPipe` để kiểm tra request.
- Sinh object key theo user, recipe và UUID để tránh trùng tên.
- Chỉ sử dụng bốn trạng thái: `pending`, `processing`, `completed` và `failed`.

### 3.2. Tích hợp Amazon S3

- Cấu hình S3 client bằng Region và thông tin xác thực từ biến môi trường.
- Gửi đúng `Content-Type` và metadata khi tạo object.
- Chỉ cấp các action S3 cần thiết cho Backend.
- Chuẩn hóa lỗi `AccessDenied`, object không tồn tại và upload bị gián đoạn.

### 3.3. Pre-signed URL

Backend tạo URL upload có thời hạn và ràng buộc object key. Sau khi trình duyệt upload trực tiếp lên S3, Frontend gọi API xác nhận; Backend kiểm tra object trước khi chuyển sang bước phân tích AI.

## 4. Kiến thức và kỹ năng đạt được

- Tổ chức module, dependency injection và validation trong NestJS.
- Sử dụng AWS SDK với S3 command và pre-signed URL.
- Thiết kế API upload an toàn và có trạng thái.
- Xử lý lỗi và dọn object khi quy trình không hoàn tất.

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

## 7. Kế hoạch tuần tiếp theo

Xây dựng giao diện upload ảnh bằng Next.js và tích hợp với các API NestJS.
