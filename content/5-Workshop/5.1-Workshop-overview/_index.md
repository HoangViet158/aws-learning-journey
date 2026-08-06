---
title: "Workshop Overview"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Introducing FoodieRecipe

FoodieRecipe is a recipe-sharing application using **Next.js** for the Frontend and **NestJS** for the Backend. Users can create accounts, publish and manage recipes, define ingredients/categories, search recipes, and upload food images.

AI assists users by:

- Moderating images and detecting food/ingredient labels with Amazon Rekognition.
- Suggesting dish names, descriptions, ingredients, and tags with Amazon Bedrock.
- Requiring users to review and edit suggestions before saving.

#### Workshop architecture

![FoodieRecipe Architecture](/images/2-Proposal/foodie-recipe-architecture.png)

| Component | Responsibility |
| --------- | -------------- |
| Next.js | Upload, preview, progress, and AI-result interface |
| NestJS | APIs, validation, image states, and AWS-service orchestration |
| S3 image bucket | `uploads/` stores originals; `delivery/` stores completed images |
| Rekognition | Label detection and content moderation |
| Bedrock | Context analysis and structured recipe suggestions |
| CloudFront | Caches and delivers images from a private S3 origin |
| EC2, Docker, Nginx | Backend runtime in the complete product architecture |
| RDS | Stores business data and image states |
| Secrets Manager, IAM | Manage secrets and access permissions |
| CloudWatch | Collects logs, metrics, and alarms |

#### End-to-end flow

1. Next.js asks NestJS to create an image record and pre-signed URL.
2. The browser uploads directly to the image bucket's `uploads/` prefix.
3. NestJS verifies the object and sets the state to `processing`.
4. Rekognition detects labels and moderates the image.
5. Bedrock produces a structured recipe suggestion.
6. An accepted image is promoted to the `delivery/` prefix in the same bucket.
7. The state becomes `completed`, and Next.js displays the CloudFront URL.
8. A failure sets the state to `failed` with an error code for retry or review.

#### Learning objectives

- Decouple upload from the Backend with pre-signed URLs.
- Design an AI workflow with validation and fallback.
- Protect S3 with Block Public Access and Origin Access Control.
- Apply least-privilege IAM and Secrets Manager.
- Observe the pipeline with CloudWatch.
