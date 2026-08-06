---
title: "Tạo S3, IAM và Secrets Manager"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

#### 1. Tạo S3 image bucket

1. Mở **Amazon S3 Console** và chọn **Create bucket**.
2. Tạo bucket theo `AWS_BUCKET_NAME`, ví dụ `my-foodie-ai-images-<account-id>`.
3. Chọn cùng Region với Backend.
4. Giữ **Block all public access** được bật.
5. Bật **Bucket Key** và server-side encryption.
6. Bật versioning nếu cần lưu lịch sử ảnh.

{{% notice note %}}
Prefix `uploads/` nhận ảnh gốc. Prefix `delivery/` chỉ chứa ảnh đã xử lý thành công và được CloudFront phân phối.
{{% /notice %}}

#### 2. Cấu hình CORS cho image bucket

Trong **Permissions → CORS**, thêm cấu hình và thay domain Frontend thực tế:

```json
[
  {
    "AllowedHeaders": ["content-type"],
    "AllowedMethods": ["PUT"],
    "AllowedOrigins": ["http://localhost:3000"],
    "ExposeHeaders": ["etag"],
    "MaxAgeSeconds": 300
  }
]
```

Không dùng `"*"` cho production origin.

#### 3. Tạo lifecycle rule

1. Mở **Management → Lifecycle rules → Create lifecycle rule**.
2. Đặt tên `cleanup-incomplete-uploads`.
3. Chọn xóa incomplete multipart uploads sau 1 ngày.
4. Nếu bật versioning, cấu hình xóa noncurrent version theo thời gian phù hợp.

#### 4. Tạo IAM role cho EC2

1. Mở **IAM → Roles → Create role**.
2. Trusted entity: **AWS service → EC2**.
3. Đặt tên `FoodieRecipeBackendRole`.
4. Gắn custom policy với ARN bucket/secret/model thực tế:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ImageObjects",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": [
        "arn:aws:s3:::my-foodie-ai-images-<account-id>/uploads/*",
        "arn:aws:s3:::my-foodie-ai-images-<account-id>/delivery/*"
      ]
    },
    {
      "Sid": "ImageAI",
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectLabels",
        "rekognition:DetectModerationLabels",
        "bedrock:InvokeModel"
      ],
      "Resource": "*"
    },
    {
      "Sid": "ReadDatabaseSecret",
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "<database-secret-arn>"
    }
  ]
}
```

Thu hẹp `Resource` cho Bedrock model khi model/Region hỗ trợ định danh ARN phù hợp.

#### 5. Tạo database secret

1. Mở **AWS Secrets Manager → Store a new secret**.
2. Chọn **Credentials for Amazon RDS database** hoặc **Other type of secret**.
3. Lưu các trường `host`, `port`, `username`, `password` và `database`.
4. Đặt tên `foodierecipe/database`.
5. Sao chép secret ARN để cập nhật IAM policy và biến `DB_SECRET_ARN`.

#### 6. Kiểm tra

```bash
aws s3api get-public-access-block --bucket my-foodie-ai-images-<account-id>
aws secretsmanager describe-secret --secret-id foodierecipe/database
```

Kết quả mong đợi: image bucket chặn public access và secret tồn tại nhưng giá trị bí mật không được in ra log.
