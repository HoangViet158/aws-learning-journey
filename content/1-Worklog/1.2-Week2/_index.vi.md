---
title: "Tuần 2: Amazon S3, RDS và bảo mật dữ liệu"
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
- Thiết kế schema PostgreSQL, RDS Security Group và cách quản lý database secret.

## 2. Kế hoạch công việc

> **Thời gian tuần 2:** Thứ 2, 29/06/2026 – Thứ 6, 03/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu S3, storage class, versioning và encryption | Hiểu cơ chế lưu trữ object |
| Thứ 3 | Tạo bucket và cấu trúc thư mục logic cho ảnh công thức | Có kho lưu trữ đúng quy ước |
| Thứ 4 | Cấu hình IAM, bucket policy, Block Public Access và CORS | Truy cập đúng phạm vi, không công khai ngoài ý muốn |
| Thứ 5 | Upload, tải xuống, xóa object và thử pre-signed URL | Hoàn thành luồng quản lý ảnh |
| Thứ 6 | Thiết kế RDS, schema dữ liệu và secret database | Hoàn thiện lớp dữ liệu và bảo mật kết nối |

## 3. Nội dung thực hiện

### 3.1. Thiết kế lưu trữ ảnh

- Tạo bucket với tên duy nhất và chọn Region phù hợp.
- Quy ước object key theo dạng `recipes/{recipeId}/{fileName}`.
- Bật versioning và server-side encryption để bảo vệ dữ liệu.
- Cấu hình lifecycle cho các phiên bản cũ nhằm giảm chi phí.

**Bucket S3 của FoodieRecipe sau khi được tạo thành công:**

![Bucket S3 FoodieRecipe được tạo thành công](/images/1-Worklog/1.2-Week2/s3-bucket-created.png)

### 3.2. Phân quyền và truy cập

- Giữ **Block Public Access** cho bucket chứa ảnh ứng dụng.
- Chỉ cấp quyền cần thiết cho Backend thông qua IAM policy.
- Cấu hình CORS cho domain Frontend và giới hạn HTTP method.
- Tạo pre-signed URL để upload hoặc truy cập object có thời hạn.

**Bật toàn bộ Block Public Access để bucket ảnh không bị truy cập công khai:**

![Cấu hình S3 Block Public Access](/images/1-Worklog/1.2-Week2/s3-block-public-access.png)

**Cấu hình CORS cho Frontend:**

```json
[
  {
    "AllowedHeaders": ["Content-Type", "x-amz-*"],
    "AllowedMethods": ["GET", "HEAD", "PUT"],
    "AllowedOrigins": [
      "http://localhost:3000",
      "https://<frontend-domain>"
    ],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

Tải file cấu hình: [cors.json](/files/1-Worklog/1.2-Week2/cors.json). Thay `<frontend-domain>` bằng domain CloudFront hoặc domain Frontend thực tế trước khi áp dụng.

**URL S3 thông thường bị từ chối do bucket không public:**

![URL S3 thông thường trả về AccessDenied](/images/1-Worklog/1.2-Week2/s3-direct-url-access-denied.png)

**Pre-signed URL cho phép truy cập object trong thời hạn quy định:**

![Truy cập ảnh thành công bằng S3 pre-signed URL](/images/1-Worklog/1.2-Week2/s3-presigned-url-success.png)

### 3.3. Metadata, RDS và Secrets Manager

Gắn metadata cần thiết như loại nội dung, chủ sở hữu và mã công thức; thống nhất trạng thái ảnh trong ứng dụng. Thiết kế PostgreSQL schema cho người dùng, công thức, nguyên liệu, danh mục, lượt thích, bình luận và ảnh; cấu hình RDS private, chỉ cho phép kết nối từ EC2 Security Group. Database credential được lưu trong Secrets Manager thay vì đưa vào source code.

**Amazon RDS for PostgreSQL của FoodieRecipe đã được tạo và ở trạng thái hoạt động:**

![Amazon RDS PostgreSQL của FoodieRecipe được tạo thành công](/images/1-Worklog/1.2-Week2/rds-database-created.png)

**Database secret đã được lưu trong AWS Secrets Manager:**

![Secret kết nối database được tạo trong AWS Secrets Manager](/images/1-Worklog/1.2-Week2/secrets-manager-created.png)

## 4. Kiến thức và kỹ năng đạt được

- Quản lý bucket và object bằng Console, CLI.
- Hiểu versioning, encryption, lifecycle và pre-signed URL.
- Phân biệt IAM policy với bucket policy.
- Biết cấu hình CORS, metadata và vòng đời object.
- Biết thiết kế RDS private, schema quan hệ và quản lý database credential bằng Secrets Manager.

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
- Mô hình PostgreSQL, RDS Security Group và database secret đã được thiết kế.

## 7. Kế hoạch tuần tiếp theo

Xây dựng Backend NestJS để upload, xác thực và quản lý ảnh trên Amazon S3.
