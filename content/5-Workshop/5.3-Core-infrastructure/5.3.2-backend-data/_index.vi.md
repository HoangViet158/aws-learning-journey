---
title: "Chuẩn bị NestJS, EC2 và Amazon RDS"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### 1. Tạo Amazon RDS

1. Mở **Amazon RDS → Create database**.
2. Chọn PostgreSQL hoặc engine mà dự án đang sử dụng.
3. Chọn template phù hợp môi trường workshop.
4. Đặt database name `foodierecipe`.
5. Chọn **Public access: No**.
6. Security Group của RDS chỉ cho phép cổng database từ Security Group của EC2.
7. Bật encryption và automated backup theo nhu cầu.

{{% notice warning %}}
Không mở cổng database cho `0.0.0.0/0`. Không ghi password RDS trực tiếp vào `.env` được commit lên Git.
{{% /notice %}}

#### 2. Chuẩn bị bảng ảnh

Schema tối thiểu:

```sql
CREATE TABLE recipe_images (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  recipe_id UUID,
  object_key VARCHAR(512) NOT NULL UNIQUE,
  status VARCHAR(20) NOT NULL,
  error_code VARCHAR(80),
  ai_result JSONB,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Giá trị `status` chỉ gồm `pending`, `processing`, `completed`, `failed`.

#### 3. Chuẩn bị EC2

1. Tạo EC2 Linux phù hợp môi trường thử nghiệm.
2. Gắn IAM role `FoodieRecipeBackendRole`.
3. Security Group chỉ mở cổng quản trị từ nguồn tin cậy và cổng API qua Nginx.
4. Cài Docker, Docker Compose plugin, Nginx và CloudWatch Agent.

#### 4. Chạy NestJS bằng Docker

Dockerfile tham khảo:

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/package*.json ./
RUN npm ci --omit=dev
COPY --from=build /app/dist ./dist
USER node
CMD ["node", "dist/main.js"]
```

Khởi chạy và kiểm tra:

```bash
cd api
docker build -t foodierecipe-backend .
docker run -d --name foodierecipe-api --restart unless-stopped -p 3001:3001 foodierecipe-backend
curl http://localhost:3001/health
```

#### 5. Cấu hình Nginx

```nginx
server {
    listen 80;

    location /api/ {
        proxy_pass http://127.0.0.1:3001/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size 2m;
    }
}
```

Reload Nginx và gọi `/api/health` để xác nhận reverse proxy hoạt động.

#### 6. Đọc secret trong NestJS

Backend dùng EC2 IAM role gọi Secrets Manager khi khởi động. Không cần lưu access key tĩnh trên máy.

Kết quả mong đợi:

- Container NestJS hoạt động sau Nginx.
- NestJS lấy được database secret.
- Kết nối RDS thành công.
- Endpoint health không in thông tin nhạy cảm.
