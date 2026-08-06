---
title: "Create S3, IAM, and Secrets Manager"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

#### 1. Create the S3 image bucket

1. Open the **Amazon S3 Console** and choose **Create bucket**.
2. Create the bucket from `AWS_BUCKET_NAME`, for example `my-foodie-ai-images-<account-id>`.
3. Select the Backend Region.
4. Keep **Block all public access** enabled.
5. Enable **Bucket Key** and server-side encryption.
6. Enable versioning if image history is required.

{{% notice note %}}
The `uploads/` prefix receives originals. The `delivery/` prefix contains only successfully processed images delivered through CloudFront.
{{% /notice %}}

#### 2. Configure image-bucket CORS

Under **Permissions → CORS**, add the following and replace the Frontend origin:

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

Do not use `"*"` for a production origin.

#### 3. Create a lifecycle rule

1. Open **Management → Lifecycle rules → Create lifecycle rule**.
2. Name it `cleanup-incomplete-uploads`.
3. Abort incomplete multipart uploads after one day.
4. If versioning is enabled, expire noncurrent versions after an appropriate period.

#### 4. Create an EC2 IAM role

1. Open **IAM → Roles → Create role**.
2. Trusted entity: **AWS service → EC2**.
3. Name it `FoodieRecipeBackendRole`.
4. Attach a custom policy using the real bucket, secret, and model identifiers:

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

Restrict the Bedrock resource to the applicable model ARN when supported by the selected model and Region.

#### 5. Create the database secret

1. Open **AWS Secrets Manager → Store a new secret**.
2. Choose **Credentials for Amazon RDS database** or **Other type of secret**.
3. Store `host`, `port`, `username`, `password`, and `database`.
4. Name the secret `foodierecipe/database`.
5. Copy its ARN into the IAM policy and `DB_SECRET_ARN` variable.

#### 6. Verify

```bash
aws s3api get-public-access-block --bucket my-foodie-ai-images-<account-id>
aws secretsmanager describe-secret --secret-id foodierecipe/database
```

Expected result: the image bucket blocks public access, and the secret exists without printing its sensitive value.
