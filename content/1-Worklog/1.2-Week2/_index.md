---
title: "Week 2: Amazon S3, RDS, and Data Security"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

## 1. Objectives

- Understand Amazon S3 buckets, objects, prefixes, and storage classes.
- Design a recipe-image bucket for FoodieRecipe.
- Control access with IAM policies, bucket policies, and CORS.
- Design metadata and object-lifecycle rules for recipe images.
- Design the PostgreSQL schema, RDS Security Group, and database-secret management.

## 2. Work plan

> **Week 2 schedule:** Monday, 29/06/2026 – Friday, 03/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study S3, storage classes, versioning, and encryption | Understand object storage |
| Tuesday | Create a bucket and logical key structure for recipe images | Establish a consistent storage layout |
| Wednesday | Configure IAM, bucket policy, Block Public Access, and CORS | Prevent unintended public access |
| Thursday | Upload, download, delete objects, and test pre-signed URLs | Complete the image-management flow |
| Friday | Design RDS, the data schema, and the database secret | Complete the data and connection-security layer |

## 3. Work completed

### 3.1. Image-storage design

- Created a uniquely named bucket in the selected Region.
- Defined object keys as `recipes/{recipeId}/{fileName}`.
- Enabled versioning and server-side encryption.
- Added lifecycle handling for older versions to control cost.

**FoodieRecipe S3 bucket after successful creation:**

![FoodieRecipe S3 bucket created successfully](/images/1-Worklog/1.2-Week2/s3-bucket-created.png)

### 3.2. Permissions and access

- Kept **Block Public Access** enabled for the application image bucket.
- Granted only the required actions to the Backend through IAM.
- Configured CORS for the Frontend domain and limited HTTP methods.
- Generated expiring pre-signed URLs for upload and access.

**Enable all Block Public Access settings to keep the image bucket private:**

![Amazon S3 Block Public Access configuration](/images/1-Worklog/1.2-Week2/s3-block-public-access.png)

**CORS configuration for the Frontend:**

```json
[
  {
    "AllowedHeaders": ["Content-Type", "x-amz-*"],
    "AllowedMethods": ["GET", "HEAD", "PUT"],
    "AllowedOrigins": [
      "http://localhost:3000",
      "https://<frontend-domain>"
    ],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

Download the configuration file: [cors.json](/files/1-Worklog/1.2-Week2/cors.json). Replace `<frontend-domain>` with the actual CloudFront or Frontend domain before applying it.

**A normal S3 URL is denied because the bucket is not public:**

![A normal S3 URL returns AccessDenied](/images/1-Worklog/1.2-Week2/s3-direct-url-access-denied.png)

**A pre-signed URL grants temporary access to the object:**

![Successful image access through an S3 pre-signed URL](/images/1-Worklog/1.2-Week2/s3-presigned-url-success.png)

### 3.3. Metadata, RDS, and Secrets Manager

Added metadata such as content type, owner, and recipe identifier and standardized image states. Designed PostgreSQL schemas for users, recipes, ingredients, categories, likes, comments, and images; kept RDS private and allowed connections only from the EC2 Security Group. Stored database credentials in Secrets Manager instead of source code.

**The FoodieRecipe Amazon RDS for PostgreSQL database was created and is available:**

![FoodieRecipe Amazon RDS PostgreSQL database created successfully](/images/1-Worklog/1.2-Week2/rds-database-created.png)

**The database secret was stored in AWS Secrets Manager:**

![Database connection secret created in AWS Secrets Manager](/images/1-Worklog/1.2-Week2/secrets-manager-created.png)

## 4. Knowledge and skills gained

- Managed buckets and objects through the Console and CLI.
- Understood versioning, encryption, lifecycle rules, and pre-signed URLs.
- Distinguished IAM policies from bucket policies.
- Configured CORS, metadata, and object lifecycle rules.
- Designed private RDS access, relational schemas, and database-secret management with Secrets Manager.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| `AccessDenied` during object operations | Checked IAM, bucket policy, and Block Public Access together |
| Browser requests blocked by CORS | Allowed only the required origin, methods, and headers |
| Possible filename collisions | Generated UUIDs and stored original names in the database |

## 6. Deliverables

- A versioned and encrypted FoodieRecipe image bucket with restricted access.
- A tested pre-signed URL upload flow.
- Defined metadata conventions and a cleanup flow for unused images.
- Defined the PostgreSQL model, RDS Security Group, and database secret.

## 7. Next-week plan

Build the NestJS Backend for uploading, validating, and managing images in Amazon S3.

## 8. References

- [Blocking public access with Amazon S3 Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [Sharing objects with Amazon S3 presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)
- [Configuring CORS for Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/enabling-cors-examples.html)
- [Amazon RDS in an Amazon VPC](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html)
- [Managing database credentials with AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
