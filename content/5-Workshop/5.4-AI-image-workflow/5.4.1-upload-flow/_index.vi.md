---
title: "Upload ảnh bằng pre-signed URL"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### 1. Cài AWS SDK cho NestJS

```bash
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

NestJS dùng credential từ EC2 IAM role. Khi chạy cục bộ, SDK dùng credential chain đã cấu hình; không hard-code access key.

#### 2. Tạo API yêu cầu upload

Request mẫu:

```json
{
  "fileName": "pho-bo.jpg",
  "contentType": "image/jpeg",
  "size": 850000,
  "recipeId": "optional-recipe-id"
}
```

Backend cần:

1. Xác thực người dùng.
2. Chỉ cho phép JPEG/PNG/WebP và giới hạn kích thước đã thống nhất.
3. Sinh `imageId` và object key an toàn; không dùng trực tiếp tên file người dùng.
4. Tạo database record ở trạng thái `pending`.
5. Trả pre-signed URL hết hạn sau khoảng 5 phút.

#### 3. Tạo pre-signed URL trong NestJS

```ts
import { Injectable } from '@nestjs/common';
import { PutObjectCommand, S3Client } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

@Injectable()
export class ImageUploadService {
  private readonly s3 = new S3Client({
    region: process.env.AWS_REGION,
  });

  async createUploadUrl(userId: string, imageId: string, contentType: string) {
    const key = `uploads/recipes/${userId}/${imageId}/original`;
    const command = new PutObjectCommand({
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: key,
      ContentType: contentType,
      Metadata: { userId, imageId },
    });

    const uploadUrl = await getSignedUrl(this.s3, command, { expiresIn: 300 });
    return { imageId, key, uploadUrl, expiresIn: 300 };
  }
}
```

{{% notice warning %}}
Client phải gửi cùng `Content-Type` đã dùng khi ký URL. Không log toàn bộ pre-signed URL vì URL chứa thông tin chữ ký tạm thời.
{{% /notice %}}

#### 4. Upload trực tiếp từ Next.js

```ts
const ticket = await api.post('/images/presign', {
  fileName: file.name,
  contentType: file.type,
  size: file.size,
});

const response = await fetch(ticket.uploadUrl, {
  method: 'PUT',
  headers: { 'Content-Type': file.type },
  body: file,
});

if (!response.ok) throw new Error('S3 upload failed');
await api.post(`/images/${ticket.imageId}/confirm`);
```

Giao diện hiển thị preview, progress và vô hiệu hóa nút upload trong lúc request đang chạy.

#### 5. Xác nhận object

Endpoint `POST /images/:imageId/confirm` dùng `HeadObjectCommand` để kiểm tra:

- Object tồn tại đúng key.
- `ContentLength` nằm trong giới hạn.
- `ContentType` hợp lệ.
- Metadata `userId` và `imageId` khớp record.

Nếu hợp lệ, đổi trạng thái thành `processing` và bắt đầu Rekognition. Nếu không hợp lệ, đổi thành `failed`, lưu `error_code` và xóa object.

#### 6. Kiểm tra

1. Upload một JPEG hợp lệ: record chuyển `pending → processing`.
2. Upload file quá lớn: API từ chối trước khi ký URL.
3. Đổi `Content-Type` khi PUT: request phải thất bại.
4. Gọi confirm với image ID của người khác: API trả 403/404.

Tài liệu tham khảo: [AWS SDK for JavaScript v3 S3 request presigner](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-s3-request-presigner/).
