---
title: "Build the Core Infrastructure"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

This section prepares the foundation for NestJS to store images, invoke AI services, and persist FoodieRecipe data.

#### Contents

1. [Create S3, IAM, and Secrets Manager](5.3.1-storage-security/)
2. [Prepare NestJS, EC2, and Amazon RDS](5.3.2-backend-data/)

#### Architecture after this section

- One private S3 image bucket with upload/delivery prefixes is ready.
- EC2 has a least-privilege IAM role.
- Database credentials are stored in Secrets Manager.
- NestJS runs in Docker behind Nginx and can connect to RDS.

{{% notice warning %}}
Create EC2/RDS only when practicing the complete product architecture. If your contribution is limited to image integration, use the existing Backend/database and still complete the S3, Rekognition, Bedrock, and CloudFront sections.
{{% /notice %}}
