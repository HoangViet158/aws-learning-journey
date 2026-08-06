---
title: "Deliver Images with Amazon CloudFront"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### 1. Prepare the S3 image bucket

1. Confirm that the bucket in `AWS_BUCKET_NAME` has **Block all public access** enabled.
2. Use **Bucket owner enforced** Object Ownership.
3. Do not enable S3 Static Website Hosting.
4. CloudFront serves only objects under `delivery/`.
5. Use versioned/hashed keys for completed images, for example:

```text
delivery/recipes/<user-id>/<recipe-id>/<image-id>-<hash>.jpg
```

#### 2. Create the CloudFront distribution

1. Open **CloudFront → Create distribution**.
2. Choose the image S3 bucket domain, not its website endpoint.
3. For Origin access, select **Origin access control settings (recommended)**.
4. Create an OAC with **Sign requests (recommended)**.
5. Set Viewer protocol policy to **Redirect HTTP to HTTPS**.
6. Allow only `GET` and `HEAD`.
7. Select a cache policy suitable for images.
8. For private delivery, create a CloudFront public key/key group, enable **Restrict viewer access**, and attach the trusted key group to the behavior.
9. Keep the corresponding private key outside the AWS Console so `api` can sign URLs; store its Base64 form in `CLOUDFRONT_PRIVATE_KEY_BASE64`.
10. Create the distribution and record its ID/domain and key pair ID.

#### 3. Update the bucket policy

Allow the CloudFront service principal to read only `delivery/` and restrict it by distribution ARN:

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

#### 4. Promote the completed image to the delivery prefix

After valid Bedrock output and user confirmation:

1. NestJS copies the image from `uploads/` to `delivery/` in `AWS_BUCKET_NAME`.
2. Set `Content-Type` and `Cache-Control: public, max-age=31536000, immutable` for a versioned/hashed key.
3. Confirm that the destination object exists.
4. Store the delivery key and CloudFront URL in the database.
5. Set the state to `completed`.
6. Delete the source image when the retention policy permits.

Never set `completed` before the copy succeeds.

#### 5. Create a CloudFront signed URL in `api`

```bash
cd api
npm install @aws-sdk/cloudfront-signer
```

When `CLOUDFRONT_DOMAIN`, `CLOUDFRONT_KEY_PAIR_ID`, and `CLOUDFRONT_PRIVATE_KEY_BASE64` are set, NestJS decodes the private key and creates a URL expiring according to `CLOUDFRONT_URL_EXPIRES_IN`:

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

When the CloudFront variables are empty locally, `api` creates an S3 pre-signed GET URL for the same object.

#### 6. Configure Next.js in `web`

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

When using `next/image`, define `NEXT_PUBLIC_CLOUDFRONT_DOMAIN` in `web/.env.local`. NestJS returns a signed URL in this form:

```text
https://<distribution-domain>/<delivery-object-key>
```

#### 7. Test caching and access

```bash
curl -I https://<distribution-domain>/<object-key>
curl -I https://my-foodie-ai-images-<account-id>.s3.<region>.amazonaws.com/delivery/<object-key>
```

Expected result:

- The CloudFront URL returns 200.
- Repeated requests show appropriate cache behavior in `X-Cache`.
- The direct S3 URL returns 403.
- Image replacements use a new key/hash instead of a distribution-wide invalidation.

Reference: [Restrict access to an Amazon S3 origin with CloudFront OAC](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html).
