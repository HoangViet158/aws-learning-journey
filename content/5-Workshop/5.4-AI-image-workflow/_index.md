---
title: "Build the AI Image Workflow"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

This section builds FoodieRecipe's primary flow from image selection to CloudFront delivery.

#### Contents

1. [Upload with a pre-signed URL](5.4.1-upload-flow/)
2. [Detect and moderate with Rekognition](5.4.2-rekognition/)
3. [Suggest recipe content with Amazon Bedrock](5.4.3-bedrock/)
4. [Deliver images with Amazon CloudFront](5.4.4-cloudfront/)

#### Workflow rules

- Use only `pending`, `processing`, `completed`, and `failed`.
- Every image has one `imageId` and a unique object key.
- Rekognition runs before Bedrock.
- Bedrock output must be validated and confirmed by the user.
- Only `completed` images are promoted to the `delivery/` prefix.
- No S3 bucket is public.
