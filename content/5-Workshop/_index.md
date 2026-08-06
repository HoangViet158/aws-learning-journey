---
title: " Workshop"
date: 2026-06-22
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building the FoodieRecipe AI Image Workflow on AWS

#### Overview

This workshop builds the image workflow for the **FoodieRecipe** recipe-sharing application. Next.js calls NestJS, uploads images directly to Amazon S3 through pre-signed URLs, uses Amazon Rekognition for detection/moderation, uses Amazon Bedrock for recipe suggestions, and uses Amazon CloudFront for faster image delivery.

The complete product architecture runs NestJS in Docker on EC2 behind Nginx, stores data in Amazon RDS, manages secrets with AWS Secrets Manager, grants access with IAM, and collects logs and metrics with Amazon CloudWatch.

{{% notice note %}}
The workshop uses four image states: `pending`, `processing`, `completed`, and `failed`.
{{% /notice %}}

#### Contents

1. [Workshop overview](5.1-workshop-overview/)
2. [Prerequisites](5.2-prerequisites/)
3. [Build the core infrastructure](5.3-core-infrastructure/)
4. [Build the AI image workflow](5.4-ai-image-workflow/)
5. [Validation, security, and monitoring](5.5-validation-monitoring/)
6. [Clean up resources](5.6-cleanup/)

#### Workshop outcomes

- An S3 image bucket with `uploads/` and `delivery/` prefixes.
- NestJS APIs for pre-signed URLs and image-state management.
- Rekognition and Bedrock integration.
- CloudFront configured with a private S3 origin.
- End-to-end validation of logs, failures, and IAM permissions.
