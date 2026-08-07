---
title: "Tuần 8: Deploy và hoàn thiện FoodieRecipe"
date: 2026-08-07T00:00:00+07:00
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

## 1. Mục tiêu

- Deploy Next.js và NestJS bằng Docker trên EC2, phục vụ qua Nginx và domain riêng.
- Hoàn thiện pipeline Next.js–NestJS–S3–Rekognition–Bedrock–CloudFront.
- Cấu hình DNS, HTTPS và kiểm tra khả năng truy cập production.
- Thu thập log ứng dụng và RDS bằng Amazon CloudWatch.
- Kiểm thử end-to-end các chức năng đăng nhập, tìm kiếm, thích, bình luận và tạo công thức bằng AI.
- Rà soát bảo mật, khả năng khởi động lại, rollback và tài liệu bàn giao.

## 2. Kế hoạch công việc

> **Thời gian tuần 8:** Thứ 2, 10/08/2026 – Thứ 6, 14/08/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Build container Next.js, chạy cùng NestJS trên EC2 và cấu hình Nginx | Frontend và Backend hoạt động trên production |
| Thứ 3 | Cấu hình DNS, HTTPS và kiểm tra CloudFront phân phối ảnh | Domain riêng và ảnh CDN hoạt động ổn định |
| Thứ 4 | Cấu hình CloudWatch Logs và kiểm tra log ứng dụng/RDS | Hệ thống có khả năng theo dõi và xử lý sự cố |
| Thứ 5 | Kiểm thử luồng đăng nhập, upload ảnh, Rekognition và Bedrock | Xác nhận pipeline AI hoàn chỉnh |
| Thứ 6 | Kiểm tra thích, bình luận, restart, rollback và hoàn thiện tài liệu | Sản phẩm sẵn sàng bàn giao |

## 3. Nội dung thực hiện

### 3.1. Deploy website với domain riêng

Frontend trong thư mục `web` sử dụng Next.js standalone output và được đóng gói bằng Docker. Khi build production, URL API và CloudFront image domain được truyền bằng build argument; container chỉ mở cổng `3000` trên loopback và được Nginx phục vụ ra ngoài qua HTTPS.

Dockerfile cần nhận CloudFront domain trước bước `pnpm build` vì biến `NEXT_PUBLIC_*` được đóng gói vào client tại build time:

```dockerfile
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}

ARG NEXT_PUBLIC_CLOUDFRONT_DOMAIN
ENV NEXT_PUBLIC_CLOUDFRONT_DOMAIN=${NEXT_PUBLIC_CLOUDFRONT_DOMAIN}

RUN pnpm build
```

```bash
cd web

docker build \
  --build-arg NEXT_PUBLIC_API_URL=https://myapps.io.vn/api \
  --build-arg NEXT_PUBLIC_CLOUDFRONT_DOMAIN=<CLOUDFRONT_DOMAIN> \
  -t foodierecipe-web:production .

docker rm -f foodierecipe-web 2>/dev/null || true
docker run -d \
  --name foodierecipe-web \
  --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  foodierecipe-web:production

curl -I http://127.0.0.1:3000
```

Nginx chuyển request giao diện tới Next.js và request `/api/` tới NestJS. Kết quả production được kiểm tra trực tiếp qua domain riêng có HTTPS:

![FoodieRecipe hoạt động trên domain myapps.io.vn](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

### 3.2. Cấu hình DNS và HTTPS bằng code

Tạo bản ghi DNS `A` cho domain `myapps.io.vn` trỏ đến Elastic IP của EC2. Ví dụ với Amazon Route 53:

```json
{
  "Comment": "Point FoodieRecipe domain to the EC2 Elastic IP",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "myapps.io.vn.",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{ "Value": "<EC2_ELASTIC_IP>" }]
      }
    }
  ]
}
```

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id <HOSTED_ZONE_ID> \
  --change-batch file://dns-change.json

dig +short myapps.io.vn
sudo certbot --nginx -d myapps.io.vn
curl -I https://myapps.io.vn
```

Nếu DNS được quản lý tại nhà cung cấp khác, tạo bản ghi `A` tương đương và giữ nguyên cấu hình Nginx/HTTPS trên EC2.

### 3.3. Xác minh CloudFront bằng code

CloudFront được dùng cho ảnh trong S3 private bucket, không thay thế container Next.js. Distribution sử dụng S3 REST origin, Origin Access Control, HTTPS và cache policy. Kiểm tra cấu hình mà không cần ảnh chụp Console:

```bash
aws cloudfront get-distribution \
  --id <DISTRIBUTION_ID> \
  --query '{Status:Distribution.Status,Domain:Distribution.DomainName,Aliases:Distribution.DistributionConfig.Aliases.Items,Origins:Distribution.DistributionConfig.Origins.Items[*].{Domain:DomainName,OAC:OriginAccessControlId},ViewerProtocol:Distribution.DistributionConfig.DefaultCacheBehavior.ViewerProtocolPolicy}'

aws cloudfront create-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --paths '/delivery/*'

curl -I '<SIGNED_CLOUDFRONT_IMAGE_URL>'
```

Kết quả mong đợi là Distribution ở trạng thái `Deployed`, URL đã ký trả về `200`, request lặp lại có header `X-Cache: Hit from cloudfront`, còn URL S3 trực tiếp trả về `403 AccessDenied`.

### 3.4. Kiểm thử end-to-end pipeline AI

**Bước 1 – Người dùng đăng nhập vào FoodieRecipe:**

![Giao diện đăng nhập FoodieRecipe](/images/1-Worklog/1.4-Week4/login-page.png)

**Bước 2 – Người dùng chọn ảnh nguyên liệu để tạo công thức:**

![Ảnh nguyên liệu được chọn](/images/1-Worklog/1.5-Week5/ai-image-selected.png)

**Bước 3 – NestJS upload ảnh lên S3 và gọi Rekognition DetectLabels:**

![Các nhãn được Rekognition nhận diện](/images/1-Worklog/1.5-Week5/rekognition-labels-result.png)

Backend lọc nhãn có độ tin cậy phù hợp, chuẩn hóa và loại bỏ nhãn trùng trước khi gửi danh sách nguyên liệu cho Bedrock.

**Bước 4 – Bedrock xử lý danh sách nguyên liệu:**

![Bedrock đang tạo công thức](/images/1-Worklog/1.6-Week6/bedrock-processing.png)

**Bước 5 – Ứng dụng hiển thị công thức được tạo:**

![Kết quả công thức do Bedrock tạo](/images/1-Worklog/1.6-Week6/bedrock-recipe-result.png)

**Bước 6 – Người dùng xem nguyên liệu và các bước thực hiện:**

![Chi tiết công thức được tạo](/images/1-Worklog/1.6-Week6/generated-recipe-details.png)

Sau luồng AI, tiếp tục kiểm tra tìm kiếm công thức, thích/bỏ thích và tạo/xóa bình luận để xác nhận Next.js, NestJS và RDS hoạt động đồng bộ trên môi trường production.

### 3.5. CloudWatch Logs và giám sát

CloudWatch có log group `foodie-recipe-log` cho log ứng dụng và `RDSOSMetrics` cho Enhanced Monitoring của RDS. Chính sách retention được đặt phù hợp để hỗ trợ xử lý sự cố mà không lưu log quá lâu.

![Các CloudWatch Log Group của FoodieRecipe](/images/1-Worklog/1.8-Week8/cloudwatch-log-groups.png)

Kiểm tra log sau khi đăng nhập, tạo công thức AI và gọi API lỗi; bảo đảm log không chứa AWS credential, JWT, URL đã ký đầy đủ hoặc giá trị bí mật từ Secrets Manager.

### 3.6. Kiểm tra vận hành và bàn giao

- Reboot EC2 và xác nhận Docker cùng Nginx tự khởi động.
- Kiểm tra health endpoint, kết nối RDS và khả năng gọi Rekognition/Bedrock sau restart.
- Thử rollback về image tag ổn định trước đó khi bản mới không vượt qua health check.
- Rà lại IAM Role theo least privilege, S3 Block Public Access, OAC và Security Group.
- Kiểm tra website qua `https://myapps.io.vn`, API, ảnh CloudFront, lượt thích và bình luận.
- Hoàn thiện kiến trúc, API contract, quy trình deploy, rollback và hướng dẫn xử lý sự cố.

## 4. Kiến thức và kỹ năng đạt được

- Deploy Next.js standalone và NestJS bằng Docker trên EC2.
- Cấu hình Nginx, DNS và HTTPS cho domain riêng.
- Kiểm tra private S3 origin và cơ chế cache của CloudFront.
- Theo dõi log ứng dụng và RDS bằng Amazon CloudWatch.
- Kiểm thử đầy đủ pipeline S3–Rekognition–Bedrock–CloudFront.
- Rà soát bảo mật, restart, rollback và tài liệu vận hành sản phẩm.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| Domain chưa phân giải tới máy chủ | Kiểm tra bản ghi `A`, nameserver và thời gian DNS propagation |
| Website HTTPS nhưng API hoặc ảnh bị chặn | Dùng HTTPS đồng nhất, kiểm tra CORS và URL production khi build Next.js |
| CloudFront trả `403` | Kiểm tra signed URL, OAC, bucket policy và object key |
| Không thấy log mới trên CloudWatch | Kiểm tra CloudWatch Agent, IAM Role, log path và Region |
| Container mới không vượt qua health check | Xem log, kiểm tra migration rồi rollback về image tag ổn định |
| Kết quả AI không đúng schema | Validate response Bedrock và ghi nhận trạng thái thất bại để thử lại có giới hạn |

## 6. Kết quả tổng kết

- FoodieRecipe hoạt động tại `https://myapps.io.vn` với Next.js và NestJS chạy bằng Docker trên EC2 sau Nginx.
- Hoàn thành đăng nhập, quản lý, tìm kiếm, thích và bình luận công thức trên môi trường production.
- Ảnh được lưu trong S3 private bucket, nhận diện bằng Rekognition và phân phối qua CloudFront.
- Bedrock tạo nội dung công thức từ các nhãn nguyên liệu đã được chuẩn hóa.
- CloudWatch thu thập log ứng dụng và RDS để hỗ trợ giám sát, kiểm tra lỗi.
- Quy trình DNS, HTTPS, deploy, restart, rollback và bàn giao đã được hoàn thiện.
