---
title: "Week 1: AWS Foundations and FoodieRecipe Design"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## 1. Objectives

- Understand cloud computing and core AWS concepts.
- Secure the AWS account with IAM and MFA.
- Configure an AWS Budget Alert for cost control.
- Analyze requirements and design the FoodieRecipe image-processing pipeline.

## 2. Work plan

> **Week 1 schedule:** Monday, 22/06/2026 – Friday, 26/06/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study Cloud Computing, Regions, Availability Zones, and core AWS services | Understand the AWS foundation |
| Tuesday | Create the account, enable MFA, and study the Shared Responsibility Model | Establish basic account protection |
| Wednesday | Create IAM users/groups, apply least privilege, and configure the AWS CLI | Access AWS without using the root user |
| Thursday | Create a Budget Alert and enable cost notifications | Establish budget monitoring |
| Friday | Analyze FoodieRecipe image features and architecture | Define the initial scope and pipeline diagram |

## 3. Work completed

### 3.1. Account setup and IAM security

- Enabled MFA for the root user and restricted root account usage.
- Created a development IAM group and attached policies following **least privilege**.
- Created an IAM user for Console and AWS CLI operations.
- Verified the active identity with `aws sts get-caller-identity`.

### 3.2. Budget management

- Created a monthly AWS cost budget.
- Configured thresholds for actual and forecasted spending.
- Registered an email recipient and verified the Budget status.

### 3.3. FoodieRecipe analysis

Defined the image-related features: selection and preview, upload, content moderation, food recognition, AI-assisted recipe suggestions, and consistently fast image display. The architecture uses Next.js for the Frontend, NestJS for the Backend, Amazon S3 for storage, Amazon Rekognition and Amazon Bedrock for AI analysis, and Amazon CloudFront for image delivery.

## 4. Knowledge and skills gained

- Distinguished IaaS, PaaS, SaaS, and the roles of AWS services in the project.
- Understood IAM users, groups, roles, and policies.
- Applied MFA and least-privilege access.
- Learned budget alerting, cost awareness, and project scoping.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| Confusing users, roles, and policies | Drew their relationships and tested read-only permissions first |
| Uncertain cost estimates | Used AWS Pricing Calculator and set a low initial budget |
| An overly broad initial scope | Prioritized core features as an MVP |

## 6. Deliverables

- An AWS account protected by MFA with a development IAM user.
- A configured AWS Budget Alert.
- FoodieRecipe requirements and an image-processing pipeline draft.

## 7. Next-week plan

Study Amazon S3 and design recipe-image upload, storage, and access control.
