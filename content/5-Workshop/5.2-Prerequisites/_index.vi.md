---
title: "Chuẩn bị môi trường"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### 1. Tài khoản và Region

1. Đăng nhập AWS bằng IAM user/role dành cho workshop, không dùng root user.
2. Chọn một Region hỗ trợ Amazon Rekognition và model Bedrock dự kiến sử dụng.
3. Ghi lại **Account ID** và **Region** để đặt tên tài nguyên.
4. Tạo AWS Budget Alert trước khi bắt đầu.

{{% notice warning %}}
Amazon Bedrock model availability và quyền truy cập phụ thuộc Region/tài khoản. Hãy xác nhận model đã được cấp quyền trước khi thực hiện phần 5.4.3.
{{% /notice %}}

#### 2. Công cụ cục bộ

Cài đặt và kiểm tra:

```bash
node --version
npm --version
aws --version
docker --version
git --version
```

Khuyến nghị sử dụng Node.js LTS, AWS CLI v2 và Docker Desktop/Engine phiên bản ổn định.

#### 3. Cấu hình AWS CLI

```bash
aws configure
aws sts get-caller-identity
aws configure get region
```

Không commit file credential, `.env` hoặc pre-signed URL vào Git.

#### 4. Chuẩn bị mã nguồn

Cấu trúc tham khảo:

```text
foodierecipe/
├── web/            # Next.js web application
├── api/            # NestJS API
├── docs/
└── .gitignore
```

Tạo file `api/.env.example` theo cấu hình của dự án:

```dotenv
# Database
DATABASE_URL=

# App
PORT=
NODE_ENV=
# Set false for HTTP deployments; set true when the API is served over HTTPS.
COOKIE_SECURE=

# JWT / auth (optional placeholders)
JWT_SECRET=
JWT_EXPIRES_IN=

# S3 bucket
AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_BUCKET_NAME=

# CloudFront private delivery (leave all values empty to use S3 presigned URLs locally)
CLOUDFRONT_DOMAIN=
CLOUDFRONT_KEY_PAIR_ID=
# Base64 of the PEM private key. Never commit the real key.
CLOUDFRONT_PRIVATE_KEY_BASE64=
CLOUDFRONT_URL_EXPIRES_IN=

# Email verification (Resend)
RESEND_API_KEY=
EMAIL_FROM=

BEDROCK_MODEL_ID=
```

{{% notice warning %}}
Các giá trị trong `.env.example` chỉ là placeholder. Không commit `.env` thật. Khi chạy `api` trên EC2, không khai báo `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`; AWS SDK sẽ dùng `FoodieRecipeBackendRole`. Đặt `COOKIE_SECURE=true` khi API được phục vụ qua HTTPS.
{{% /notice %}}

- `DATABASE_URL` sử dụng PostgreSQL local ở cổng `5433`.
- `CLOUDFRONT_*` để trống khi chạy local; `api` sẽ trả S3 pre-signed GET URL.
- `CLOUDFRONT_PRIVATE_KEY_BASE64` là private key đã mã hóa Base64, không phải nội dung PEM thuần.
- `RESEND_API_KEY` và `EMAIL_FROM` chỉ cần khi bật xác minh email.

#### 5. Quyền cần thiết

IAM principal thực hiện workshop cần quyền tạo/quản lý các tài nguyên được sử dụng: S3, CloudFront, EC2, IAM role, RDS, Secrets Manager, CloudWatch Logs, Rekognition và Bedrock.

{{% notice note %}}
Không dùng policy `AdministratorAccess` cho ứng dụng. Ở phần 5.3.1, bạn sẽ tạo role riêng cho EC2 theo nguyên tắc least privilege.
{{% /notice %}}

#### 6. Quy ước tài nguyên

| Tài nguyên | Tên gợi ý |
| ---------- | ---------- |
| Image bucket | `my-foodie-ai-images` hoặc tên duy nhất tương đương |
| EC2 role | `FoodieRecipeBackendRole` |
| Secret | `foodierecipe/database` |
| Log group | `/foodierecipe/api` |

Sau khi hoàn tất, chuyển sang [Xây dựng hạ tầng cốt lõi](../5.3-core-infrastructure/).
