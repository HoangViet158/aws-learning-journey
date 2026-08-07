---
title: "Tuần 7: Triển khai AWS và Amazon CloudFront"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

## 1. Mục tiêu

- Phân phối ảnh FoodieRecipe từ S3 thông qua CloudFront.
- Giữ bucket private và kiểm soát quyền đọc từ CloudFront.
- Thiết kế cache policy phù hợp với ảnh có phiên bản.
- Đo và cải thiện tốc độ tải ảnh trên Next.js.
- Tạo Amazon RDS private, EC2, Security Group, IAM Role và Secrets Manager.
- Deploy NestJS bằng Docker, chạy migration và cấu hình Nginx reverse proxy.

## 2. Kế hoạch công việc

> **Thời gian tuần 7:** Thứ 2, 03/08/2026 – Thứ 6, 07/08/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tạo RDS private, DB subnet group, Security Group và secret | Database sẵn sàng cho Backend |
| Thứ 3 | Tạo EC2, gắn IAM Role và cài Docker/Nginx/CloudWatch Agent | Máy chủ sẵn sàng triển khai |
| Thứ 4 | Build container, chạy migration và cấu hình Nginx | NestJS hoạt động trên EC2 |
| Thứ 5 | Tạo CloudFront với S3 origin và Origin Access Control | Ảnh private được tải qua CDN |
| Thứ 6 | Tích hợp URL, đo cache hit/latency và kiểm tra restart | Xác nhận deployment và hiệu năng |

## 3. Nội dung thực hiện

### 3.1. Kiến trúc triển khai tổng thể

Kiến trúc FoodieRecipe kết hợp Next.js, NestJS và các dịch vụ AWS để cung cấp ứng dụng hoàn chỉnh. CloudFront phân phối nội dung và ảnh từ S3; Next.js gọi API NestJS chạy trong Docker phía sau Nginx trên EC2. Backend kết nối PostgreSQL trên RDS, gọi Rekognition để nhận diện nguyên liệu và Bedrock để tạo công thức. Secrets Manager quản lý bí mật, IAM kiểm soát quyền truy cập và CloudWatch thu thập metric, log của hệ thống.

![Kiến trúc triển khai FoodieRecipe trên AWS](/images/1-Worklog/1.7-Week7/foodierecipe-aws-architecture.png)

Luồng chính trong sơ đồ:

1. Người dùng truy cập nội dung qua Amazon CloudFront.
2. CloudFront lấy và cache nội dung từ Amazon S3.
3. Next.js gửi request nghiệp vụ tới NestJS trên EC2.
4. Ảnh được tải lên bucket S3 private theo luồng upload có kiểm soát.
5. Backend xử lý thông tin object và lưu metadata của ảnh.
6. NestJS đọc và ghi dữ liệu FoodieRecipe trên Amazon RDS PostgreSQL.
7. Amazon Bedrock tạo nội dung công thức từ danh sách nguyên liệu.
8. Amazon Rekognition nhận diện nhãn và nguyên liệu trong ảnh.
9. EC2 lấy bí mật ứng dụng từ AWS Secrets Manager thông qua IAM Role.

### 3.2. Tạo EC2 và gắn IAM Role

Tạo EC2 Linux trong public subnet, gắn Elastic IP, IAM Role `DeployEC2` và các Security Group cần thiết. Chỉ mở SSH `22` từ IP quản trị, mở HTTP `80`/HTTPS `443` cho người dùng và không công khai cổng NestJS `3001`. RDS chỉ nhận PostgreSQL `5432` từ Security Group của Backend.

**EC2 FoodieRecipe hoạt động và vượt qua toàn bộ status check:**

![EC2 FoodieRecipe ở trạng thái Running](/images/1-Worklog/1.7-Week7/ec2-instance-running.png)

**IAM Role được gắn với EC2 để ứng dụng nhận temporary credential:**

![IAM Role và Security Group của EC2 FoodieRecipe](/images/1-Worklog/1.7-Week7/ec2-iam-role.png)

Sau khi SSH vào máy chủ, kiểm tra IAM identity và cài công cụ triển khai:

```bash
aws sts get-caller-identity

sudo dnf update -y
sudo dnf install -y git docker nginx jq
sudo systemctl enable --now docker
sudo systemctl enable --now nginx
sudo usermod -aG docker ec2-user
```

Đăng xuất rồi SSH lại để quyền Docker có hiệu lực. IAM Role chỉ cấp quyền cần thiết cho S3, Rekognition, Bedrock, Secrets Manager và CloudWatch; không lưu access key trên EC2.

### 3.3. Build, migration và chạy NestJS bằng Docker

Dockerfile của `api` sử dụng Node.js 22, cài dependency bằng pnpm, generate Prisma Client, build NestJS và tự chạy `prisma migrate deploy` trước khi khởi động `node dist/main.js`.

```bash
git clone https://github.com/<owner>/<repository>.git foodierecipe
cd foodierecipe/api
docker build -t foodierecipe-api:week7 .
```

Không tạo `.env.production`. Cấu hình không nhạy cảm được lưu trong Systems Manager Parameter Store, còn thông tin nhạy cảm được lưu dưới dạng JSON trong Secrets Manager:

```text
Parameter Store
├── /foodierecipe/prod/PORT
├── /foodierecipe/prod/AWS_BUCKET_NAME
├── /foodierecipe/prod/BEDROCK_MODEL_ID
├── /foodierecipe/prod/CLOUDFRONT_DOMAIN
├── /foodierecipe/prod/CLOUDFRONT_KEY_PAIR_ID
└── /foodierecipe/prod/CLOUDFRONT_URL_EXPIRES_IN

Secrets Manager
├── prod/foodie-recipe/db
│   └── DATABASE_URL
└── prod/foodie-recipe/app
    ├── JWT_SECRET
    └── CLOUDFRONT_PRIVATE_KEY_BASE64
```

IAM Role `DeployEC2` cần `ssm:GetParameter`, `secretsmanager:GetSecretValue` và, nếu secret dùng customer managed KMS key, `kms:Decrypt` đối với đúng ARN được sử dụng. Script triển khai lấy giá trị bằng temporary credential của IAM Role, truyền trực tiếp vào container rồi xóa biến tạm; không tạo file chứa bí mật trên ổ đĩa:

```bash
#!/usr/bin/env bash
set -euo pipefail

REGION="ap-southeast-1"
PARAMETER_PATH="/foodierecipe/prod"

get_parameter() {
  aws ssm get-parameter \
    --region "$REGION" \
    --name "$PARAMETER_PATH/$1" \
    --query 'Parameter.Value' \
    --output text
}

DB_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id "prod/foodie-recipe/db" \
  --query SecretString \
  --output text)

APP_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id "prod/foodie-recipe/app" \
  --query SecretString \
  --output text)

DATABASE_URL=$(jq -r '.DATABASE_URL' <<< "$DB_SECRET")
JWT_SECRET=$(jq -r '.JWT_SECRET' <<< "$APP_SECRET")
CLOUDFRONT_PRIVATE_KEY_BASE64=$(jq -r '.CLOUDFRONT_PRIVATE_KEY_BASE64' <<< "$APP_SECRET")

docker rm -f foodierecipe-api 2>/dev/null || true
docker run -d \
  --name foodierecipe-api \
  --restart unless-stopped \
  -p 127.0.0.1:3001:3001 \
  -e "DATABASE_URL=$DATABASE_URL" \
  -e "PORT=$(get_parameter PORT)" \
  -e "NODE_ENV=production" \
  -e "COOKIE_SECURE=true" \
  -e "JWT_SECRET=$JWT_SECRET" \
  -e "JWT_EXPIRES_IN=7d" \
  -e "AWS_REGION=$REGION" \
  -e "AWS_BUCKET_NAME=$(get_parameter AWS_BUCKET_NAME)" \
  -e "BEDROCK_MODEL_ID=$(get_parameter BEDROCK_MODEL_ID)" \
  -e "CLOUDFRONT_DOMAIN=$(get_parameter CLOUDFRONT_DOMAIN)" \
  -e "CLOUDFRONT_KEY_PAIR_ID=$(get_parameter CLOUDFRONT_KEY_PAIR_ID)" \
  -e "CLOUDFRONT_PRIVATE_KEY_BASE64=$CLOUDFRONT_PRIVATE_KEY_BASE64" \
  -e "CLOUDFRONT_URL_EXPIRES_IN=$(get_parameter CLOUDFRONT_URL_EXPIRES_IN)" \
  foodierecipe-api:week7

unset DB_SECRET APP_SECRET DATABASE_URL JWT_SECRET CLOUDFRONT_PRIVATE_KEY_BASE64

docker ps
docker logs --tail 100 foodierecipe-api
curl http://127.0.0.1:3001/api
```

Log container phải thể hiện migration hoàn tất và NestJS khởi động thành công. Kiểm tra thêm kết nối RDS bằng API đăng nhập, tạo công thức hoặc Prisma query thay vì mở cổng database ra Internet.

### 3.4. Cấu hình Nginx reverse proxy

Tạo `/etc/nginx/conf.d/foodierecipe.conf`:

```nginx
server {
    listen 80;
    server_name _;

    client_max_body_size 10m;

    location /api/ {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Kiểm tra cấu hình và gọi API qua Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
curl http://127.0.0.1/api
curl http://<EC2_PUBLIC_IP>/api
```

### 3.5. Tạo CloudFront OAC và bucket policy

Giữ S3 Block Public Access. Tạo file `oac-config.json` và Origin Access Control bằng AWS CLI:

```json
{
  "Name": "foodierecipe-s3-oac",
  "Description": "Private S3 origin for FoodieRecipe images",
  "SigningProtocol": "sigv4",
  "SigningBehavior": "always",
  "OriginAccessControlOriginType": "s3"
}
```

```bash
aws cloudfront create-origin-access-control \
  --origin-access-control-config file://oac-config.json
```

Distribution sử dụng S3 REST origin, OAC vừa tạo, **Redirect HTTP to HTTPS**, chỉ cho phép `GET`/`HEAD` và managed cache policy `CachingOptimized`. Bucket policy chỉ cho distribution tương ứng đọc object:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontReadOnly",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-foodie-ai-images/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<AWS_ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
        }
      }
    }
  ]
}
```

Tạo RSA key pair cho CloudFront signed URL; private key không được commit:

```bash
openssl genrsa -out cloudfront_private_key.pem 2048
openssl rsa -pubout \
  -in cloudfront_private_key.pem \
  -out cloudfront_public_key.pem
```

Upload public key, tạo trusted key group và gắn vào image behavior. Backend dùng các biến `CLOUDFRONT_DOMAIN`, `CLOUDFRONT_KEY_PAIR_ID`, `CLOUDFRONT_PRIVATE_KEY_BASE64` và `CLOUDFRONT_URL_EXPIRES_IN` để tạo URL có thời hạn.

### 3.6. Kiểm tra URL và cache

```bash
# URL CloudFront đã ký phải trả về 200.
curl -I "<CLOUDFRONT_SIGNED_URL>"

# Xóa Signature/Key-Pair-Id hoặc dùng direct S3 URL phải trả về 403.
curl -I "https://<DISTRIBUTION_DOMAIN>/<OBJECT_KEY>"
curl -I "https://my-foodie-ai-images.s3.ap-southeast-1.amazonaws.com/<OBJECT_KEY>"

# Gọi lại URL hợp lệ và kiểm tra X-Cache/Age để xác nhận edge cache.
curl -I "<CLOUDFRONT_SIGNED_URL>"
```

Object key chứa timestamp hoặc hash nên có thể dùng TTL dài mà không hiển thị ảnh cũ. Chỉ tạo invalidation khi không thể thay đổi object key.

## 4. Kiến thức và kỹ năng đạt được

- Hiểu CloudFront distribution, origin, behavior và edge cache.
- Bảo vệ S3 origin bằng Origin Access Control.
- Thiết kế cache key, TTL và versioned object key.
- Đo cache hit/miss và thời gian tải ảnh.
- Triển khai RDS, EC2, Docker, Nginx, IAM Role và Secrets Manager.
- Chạy migration, health check và kiểm tra container tự khởi động lại.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| CloudFront trả 403 dù object tồn tại | Kiểm tra OAC, bucket policy và object key |
| Ảnh cũ còn trong cache | Dùng key có version/hash hoặc invalidation có chọn lọc |
| Cache hit thấp | Giảm biến thể query/header trong cache key |
| Container không khởi động sau deploy | Kiểm tra migration, biến môi trường và `docker logs` |
| API chạy local nhưng không gọi được từ ngoài | Kiểm tra Nginx, Security Group và cổng `3001` chỉ bind loopback |

## 6. Kết quả đầu ra

- CloudFront distribution đọc ảnh từ bucket S3 private.
- URL ảnh CloudFront được tích hợp vào NestJS và Next.js.
- Cache policy và object-versioning strategy giúp cải thiện tốc độ truy cập.
- Backend NestJS đã được deploy lên EC2, kết nối RDS và phục vụ qua Nginx.

## 7. Kế hoạch tuần tiếp theo

Deploy Frontend Next.js, cấu hình CloudWatch, tích hợp toàn bộ pipeline, kiểm thử và hoàn thiện tài liệu.
