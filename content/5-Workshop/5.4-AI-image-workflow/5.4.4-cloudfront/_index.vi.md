---
title: "Phân phối ảnh với Amazon CloudFront"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### 1. Chuẩn bị S3 image bucket

1. Xác nhận bucket trong `AWS_BUCKET_NAME` bật **Block all public access**.
2. Object Ownership dùng **Bucket owner enforced**.
3. Không bật S3 Static Website Hosting.
4. CloudFront chỉ phân phối object dưới prefix `delivery/`.
5. Ảnh hoàn tất sử dụng object key có version/hash, ví dụ:

```text
delivery/recipes/<user-id>/<recipe-id>/<image-id>-<hash>.jpg
```

#### 2. Tạo CloudFront distribution

1. Mở **CloudFront → Create distribution**.
2. Origin domain: chọn S3 image bucket, không chọn website endpoint.
3. Origin access: chọn **Origin access control settings (recommended)**.
4. Tạo OAC mới, chọn **Sign requests (recommended)**.
5. Viewer protocol policy: **Redirect HTTP to HTTPS**.
6. Allowed methods: `GET`, `HEAD`.
7. Chọn cache policy phù hợp nội dung ảnh.
8. Với private delivery, tạo CloudFront public key/key group, bật **Restrict viewer access** và gắn trusted key group vào behavior.
9. Giữ private key tương ứng ngoài AWS Console để `api` ký URL; lưu Base64 của key trong `CLOUDFRONT_PRIVATE_KEY_BASE64`.
10. Tạo distribution và ghi lại distribution ID/domain cùng key pair ID.

#### 3. Cập nhật bucket policy

Cho phép CloudFront service principal chỉ đọc prefix `delivery/`, giới hạn bằng distribution ARN:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontReadOnly",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-foodie-ai-images-<account-id>/delivery/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
        }
      }
    }
  ]
}
```

#### 4. Đưa ảnh hoàn tất sang delivery prefix

Sau khi Bedrock output hợp lệ và người dùng xác nhận:

1. NestJS copy ảnh từ `uploads/` sang `delivery/` trong `AWS_BUCKET_NAME`.
2. Đặt `Content-Type` và `Cache-Control: public, max-age=31536000, immutable` cho object có key version/hash.
3. Xác nhận object đích tồn tại.
4. Lưu delivery key và CloudFront URL vào database.
5. Chuyển trạng thái thành `completed`.
6. Xóa ảnh nguồn khi quy tắc lưu trữ cho phép.

Không đặt `completed` trước khi copy thành công.

#### 5. Tạo CloudFront signed URL trong `api`

```bash
cd api
npm install @aws-sdk/cloudfront-signer
```

Khi `CLOUDFRONT_DOMAIN`, `CLOUDFRONT_KEY_PAIR_ID` và `CLOUDFRONT_PRIVATE_KEY_BASE64` có giá trị, NestJS giải mã private key và tạo URL hết hạn theo `CLOUDFRONT_URL_EXPIRES_IN`:

```ts
import { getSignedUrl } from '@aws-sdk/cloudfront-signer';

const privateKey = Buffer.from(
  process.env.CLOUDFRONT_PRIVATE_KEY_BASE64!,
  'base64',
).toString('utf8');

const signedUrl = getSignedUrl({
  url: `https://${process.env.CLOUDFRONT_DOMAIN}/${deliveryKey}`,
  keyPairId: process.env.CLOUDFRONT_KEY_PAIR_ID!,
  privateKey,
  dateLessThan: new Date(
    Date.now() + Number(process.env.CLOUDFRONT_URL_EXPIRES_IN ?? 300) * 1000,
  ).toISOString(),
});
```

Nếu các biến CloudFront để trống ở local, `api` tạo S3 pre-signed GET URL cho cùng object.

#### 6. Cấu hình Next.js trong `web`

```ts
// web/next.config.ts
const nextConfig = {
  images: {
    remotePatterns: [{
      protocol: 'https',
      hostname: process.env.NEXT_PUBLIC_CLOUDFRONT_DOMAIN,
    }],
  },
};

export default nextConfig;
```

Nếu dùng `next/image`, khai báo `NEXT_PUBLIC_CLOUDFRONT_DOMAIN` trong `web/.env.local`. NestJS trả URL đã ký dạng:

```text
https://<distribution-domain>/<delivery-object-key>
```

#### 7. Kiểm tra cache và quyền

```bash
curl -I https://<distribution-domain>/<object-key>
curl -I https://my-foodie-ai-images-<account-id>.s3.<region>.amazonaws.com/delivery/<object-key>
```

Kết quả mong đợi:

- CloudFront URL trả 200.
- Request lặp lại có header cache (`X-Cache`) phù hợp.
- Direct S3 URL trả 403.
- Khi thay ảnh, dùng key/hash mới thay vì invalidation toàn distribution.

Tài liệu tham khảo: [Restrict access to an Amazon S3 origin with CloudFront OAC](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html).
