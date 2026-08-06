---
title: "Week 4: Next.js Frontend and Image Upload Experience"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

## 1. Objectives

- Build image selection, preview, and upload interfaces with Next.js.
- Integrate the NestJS pre-signed URL flow.
- Clearly present upload progress, processing states, and errors.
- Optimize the experience for desktop and mobile devices.

## 2. Work plan

> **Week 4 schedule:** Monday, 13/07/2026 – Friday, 17/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Design the upload component and UI states | Establish a clear user flow |
| Tuesday | Add drag-and-drop, preview, and client validation | Catch errors before sending a file |
| Wednesday | Integrate the pre-signed URL API and direct S3 upload | Keep image bytes out of NestJS |
| Thursday | Display progress, retry, and processing states | Let users follow the complete workflow |
| Friday | Test responsiveness, accessibility, and network errors | Stabilize the interface across devices |

## 3. Work completed

### 3.1. Upload component

- Supported file selection and drag-and-drop.
- Displayed preview, filename, size, and replace/remove controls.
- Validated format and size before calling the Backend.
- Released object URLs when no longer needed to avoid memory leaks.

### 3.2. NestJS and S3 integration

- Called NestJS to create an image record and receive a pre-signed URL.
- Uploaded directly to S3 with the correct `Content-Type`.
- Called the confirmation API after a successful upload.
- Kept AWS credentials out of source code and the browser.

### 3.3. Processing states

The interface represents four states: `pending`, `processing`, `completed`, and `failed`. After network errors, users can retry in a controlled way without creating duplicate objects.

## 4. Knowledge and skills gained

- Built interactive Next.js and TypeScript components.
- Managed browser files, previews, and upload progress.
- Integrated Next.js with NestJS and S3 pre-signed URLs.
- Improved accessibility and error-handling experience.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| The UI reported success after a failed upload | Confirmed only after S3 returned a successful response |
| Users could trigger upload repeatedly | Disabled controls during processing and reused an idempotent image ID |
| Preview URLs consumed memory | Revoked object URLs after replacement or component unmount |

## 6. Deliverables

- A Next.js image-upload component with preview and validation.
- Direct S3 upload through a pre-signed URL.
- Progress, retry, and processing states synchronized with NestJS.

## 7. Next-week plan

Integrate Amazon Rekognition for food-image detection and moderation.
