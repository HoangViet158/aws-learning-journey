---
title: "Week 3: NestJS Backend and Amazon S3"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

## 1. Objectives

- Build the image-processing module in the NestJS Backend.
- Integrate the AWS SDK for JavaScript with Amazon S3.
- Validate image format, size, and ownership.
- Build a direct-upload flow with pre-signed URLs.

## 2. Work plan

> **Week 3 schedule:** Monday, 06/07/2026 – Friday, 10/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Design the image module, controller, service, and DTOs | Establish a clear Backend structure |
| Tuesday | Integrate the S3 client and server-side upload API | Upload images under the correct object key |
| Wednesday | Add file type, size, and metadata validation | Reject invalid files before storage |
| Thursday | Build pre-signed URL and upload-confirmation APIs | Keep image bytes out of the Backend path |
| Friday | Build image read, replacement, deletion, and error handling | Complete the image lifecycle |

## 3. Work completed

### 3.1. NestJS image module

- Separated `ImageModule`, `ImageController`, and `ImageService`.
- Used DTOs and `ValidationPipe` for request validation.
- Generated object keys from the user, recipe, and a UUID.
- Used only four states: `pending`, `processing`, `completed`, and `failed`.

### 3.2. Amazon S3 integration

- Configured the S3 client with the Region and environment-based credentials.
- Sent the correct `Content-Type` and metadata when creating objects.
- Granted only the required S3 actions to the Backend.
- Standardized errors for access denial, missing objects, and interrupted uploads.

### 3.3. Pre-signed URLs

The Backend creates an expiring upload URL bound to an object key. After the browser uploads directly to S3, the Frontend calls a confirmation API; the Backend checks the object before starting AI analysis.

## 4. Knowledge and skills gained

- Organized NestJS modules, dependency injection, and validation.
- Used the AWS SDK with S3 commands and pre-signed URLs.
- Designed a secure, stateful image-upload API.
- Handled errors and cleaned up incomplete objects.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| A file could spoof its MIME type | Checked MIME type, extension, and file signature |
| The URL expired during upload | Allowed a new URL while keeping the same image record |
| Upload completed without confirmation | Kept a `pending` state and cleaned up expired objects |

## 6. Deliverables

- A NestJS image module integrated with Amazon S3.
- Server-side upload and pre-signed URL flows.
- Validation, image states, and deletion handling.

## 7. Next-week plan

Build the Next.js image-upload interface and integrate it with the NestJS APIs.
