---
title: "Tuần 2: Amazon S3 và lưu trữ nội dung"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

## 1. Mục tiêu

- Hiểu cấu trúc bucket, object, prefix và các lớp lưu trữ Amazon S3.
- Thiết kế bucket lưu ảnh công thức cho FoodieRecipe.
- Kiểm soát truy cập bằng IAM policy, bucket policy và CORS.
- Thiết kế metadata và vòng đời object cho ảnh công thức.

## 2. Kế hoạch công việc

> **Thời gian tuần 2:** Thứ 2, 29/06/2026 – Thứ 6, 03/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu S3, storage class, versioning và encryption | Hiểu cơ chế lưu trữ object |
| Thứ 3 | Tạo bucket và cấu trúc thư mục logic cho ảnh công thức | Có kho lưu trữ đúng quy ước |
| Thứ 4 | Cấu hình IAM, bucket policy, Block Public Access và CORS | Truy cập đúng phạm vi, không công khai ngoài ý muốn |
| Thứ 5 | Upload, tải xuống, xóa object và thử pre-signed URL | Hoàn thành luồng quản lý ảnh |
| Thứ 6 | Kiểm tra metadata, versioning, lifecycle và quy trình xóa ảnh | Hoàn thiện quy tắc quản lý vòng đời ảnh |

## 3. Nội dung thực hiện

### 3.1. Thiết kế lưu trữ ảnh

- Tạo bucket với tên duy nhất và chọn Region phù hợp.
- Quy ước object key theo dạng `recipes/{recipeId}/{fileName}`.
- Bật versioning và server-side encryption để bảo vệ dữ liệu.
- Cấu hình lifecycle cho các phiên bản cũ nhằm giảm chi phí.

### 3.2. Phân quyền và truy cập

- Giữ **Block Public Access** cho bucket chứa ảnh ứng dụng.
- Chỉ cấp quyền cần thiết cho Backend thông qua IAM policy.
- Cấu hình CORS cho domain Frontend và giới hạn HTTP method.
- Tạo pre-signed URL để upload hoặc truy cập object có thời hạn.

### 3.3. Metadata và vòng đời ảnh

Gắn metadata cần thiết như loại nội dung, chủ sở hữu và mã công thức; thống nhất trạng thái ảnh trong ứng dụng. Thử versioning, lifecycle và quy trình xóa object để hạn chế ảnh trùng hoặc ảnh không còn được sử dụng.

## 4. Kiến thức và kỹ năng đạt được

- Quản lý bucket và object bằng Console, CLI.
- Hiểu versioning, encryption, lifecycle và pre-signed URL.
- Phân biệt IAM policy với bucket policy.
- Biết cấu hình CORS, metadata và vòng đời object.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| Lỗi `AccessDenied` khi thao tác object | Kiểm tra đồng thời IAM policy, bucket policy và Block Public Access |
| Trình duyệt chặn request do CORS | Chỉ định đúng origin, method và header cần thiết |
| Tên file có nguy cơ trùng | Sinh UUID và lưu tên gốc trong cơ sở dữ liệu |

## 6. Kết quả đầu ra

- Bucket ảnh FoodieRecipe có versioning, encryption và quyền truy cập phù hợp.
- Luồng upload bằng pre-signed URL đã được thử nghiệm.
- Quy ước metadata và quy trình dọn ảnh không còn sử dụng đã được xác định.

## 7. Kế hoạch tuần tiếp theo

Xây dựng Backend NestJS để upload, xác thực và quản lý ảnh trên Amazon S3.
