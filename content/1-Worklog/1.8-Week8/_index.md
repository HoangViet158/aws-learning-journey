---
title: "Week 8: Image Pipeline Integration and Completion"
date: 2026-08-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

## 1. Objectives

- Complete the Next.js–NestJS–S3–Rekognition–Bedrock–CloudFront pipeline.
- Test functionality, security, performance, and failure cases.
- Review IAM permissions and the cost of each AWS service.
- Complete the architecture diagram, API documentation, and work report.

## 2. Work plan

> **Week 8 schedule:** Monday, 10/08/2026 – Friday, 14/08/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Test end-to-end from image selection to AI results | Confirm the primary flow works correctly |
| Tuesday | Test invalid files, timeouts, retries, and malformed AI output | Make the pipeline resilient |
| Wednesday | Review IAM, S3 policy, OAC, CORS, and sensitive data | Reduce permissions and security risks |
| Thursday | Measure latency, cache hits, and Rekognition/Bedrock cost | Identify optimization opportunities |
| Friday | Complete documentation, diagrams, demo, and report | Prepare the work for handover |

## 3. Work completed

### 3.1. Complete pipeline

1. Next.js asks NestJS to create an image record and pre-signed URL.
2. The browser uploads directly to S3 and confirms with NestJS.
3. NestJS invokes Rekognition for label detection and moderation.
4. The accepted image and labels are sent to Bedrock for recipe suggestions.
5. The user reviews the result, and the image is displayed through CloudFront.

### 3.2. Testing and error handling

- Tested invalid formats, oversized files, interrupted uploads, and expired URLs.
- Tested images rejected or marked for review by Rekognition.
- Simulated Bedrock timeout, schema errors, and exhausted retries.
- Confirmed obsolete images were cleaned up and states did not remain stuck.

### 3.3. Security and cost

- Granted the Backend only the required S3, Rekognition, and Bedrock permissions.
- Kept S3 private and delivered user-facing images through CloudFront.
- Avoided logging AWS credentials, complete pre-signed URLs, or sensitive data.
- Cached AI results, limited retries, and tracked service invocation counts.

### 3.4. Handover documentation

Completed the sequence diagram, image-state documentation, API contract, environment variables, error table, testing process, and troubleshooting guide. Prepared a demo from image upload to recognition results and the CloudFront URL.

## 4. Knowledge and skills gained

- Built an end-to-end image flow between Next.js and NestJS.
- Integrated S3, Rekognition, Bedrock, and CloudFront with clear responsibilities.
- Designed an AI workflow with validation, fallback, and human review.
- Evaluated the security, performance, and cost of an image-processing pipeline.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| State became inconsistent across asynchronous stages | Used one image ID and an explicit state machine |
| AI results were difficult to reproduce | Recorded model ID, prompt version, input hash, and normalized response |
| It was difficult to identify the slow stage | Timed upload, Rekognition, Bedrock, and CloudFront separately |

## 6. Final outcomes

- Completed image upload and management with Next.js, NestJS, and S3.
- Rekognition performs detection/moderation, while Bedrock suggests recipe content.
- CloudFront delivers S3 images with caching for faster access.
- The pipeline includes validation, error handling, access control, and complete documentation.
