---
title: "Upload with a Pre-Signed URL"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### 1. Install the AWS SDK packages for NestJS

```bash
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

NestJS receives credentials from the EC2 IAM role. During local development, the SDK uses the configured credential chain; never hard-code access keys.

#### 2. Create the upload-request API

Example request:

```json
{
  "fileName": "pho-bo.jpg",
  "contentType": "image/jpeg",
  "size": 850000,
  "recipeId": "optional-recipe-id"
}
```

The Backend must:

1. Authenticate the user.
2. Allow only JPEG/PNG/WebP and enforce the agreed size limit.
3. Generate a safe `imageId` and object key without trusting the user filename.
4. Create a database record in `pending`.
5. Return a pre-signed URL expiring in approximately five minutes.

#### 3. Create the pre-signed URL in NestJS

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
The client must send the same `Content-Type` used when signing. Never log the complete pre-signed URL because it contains temporary signature data.
{{% /notice %}}

#### 4. Upload directly from Next.js

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

The interface shows preview/progress and disables repeated upload actions while the request is running.

#### 5. Confirm the object

The `POST /images/:imageId/confirm` endpoint uses `HeadObjectCommand` to verify:

- The object exists under the expected key.
- `ContentLength` is within the limit.
- `ContentType` is allowed.
- The `userId` and `imageId` metadata match the record.

If valid, change the state to `processing` and begin Rekognition. Otherwise set `failed`, store an `error_code`, and delete the object.

#### 6. Test

1. Upload a valid JPEG: the record moves `pending → processing`.
2. Upload an oversized file: the API rejects it before signing.
3. Change `Content-Type` during PUT: the request should fail.
4. Confirm another user's image ID: the API returns 403/404.

Reference: [AWS SDK for JavaScript v3 S3 request presigner](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-s3-request-presigner/).
