---
title: "Week 2: Amazon S3 and Content Storage"
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

## 2. Work plan

> **Week 2 schedule:** Monday, 29/06/2026 – Friday, 03/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study S3, storage classes, versioning, and encryption | Understand object storage |
| Tuesday | Create a bucket and logical key structure for recipe images | Establish a consistent storage layout |
| Wednesday | Configure IAM, bucket policy, Block Public Access, and CORS | Prevent unintended public access |
| Thursday | Upload, download, delete objects, and test pre-signed URLs | Complete the image-management flow |
| Friday | Test metadata, versioning, lifecycle, and image deletion | Complete image-lifecycle rules |

## 3. Work completed

### 3.1. Image-storage design

- Created a uniquely named bucket in the selected Region.
- Defined object keys as `recipes/{recipeId}/{fileName}`.
- Enabled versioning and server-side encryption.
- Added lifecycle handling for older versions to control cost.

### 3.2. Permissions and access

- Kept **Block Public Access** enabled for the application image bucket.
- Granted only the required actions to the Backend through IAM.
- Configured CORS for the Frontend domain and limited HTTP methods.
- Generated expiring pre-signed URLs for upload and access.

### 3.3. Image metadata and lifecycle

Added metadata such as content type, owner, and recipe identifier, and defined consistent image states in the application. Tested versioning, lifecycle rules, and object deletion to reduce duplicates and abandoned images.

## 4. Knowledge and skills gained

- Managed buckets and objects through the Console and CLI.
- Understood versioning, encryption, lifecycle rules, and pre-signed URLs.
- Distinguished IAM policies from bucket policies.
- Configured CORS, metadata, and object lifecycle rules.

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

## 7. Next-week plan

Build the NestJS Backend for uploading, validating, and managing images in Amazon S3.
