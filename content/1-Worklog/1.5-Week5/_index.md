---
title: "Week 5: Image Recognition with Amazon Rekognition"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## 1. Objectives

- Integrate Amazon Rekognition with images stored in S3.
- Detect labels related to food and ingredients.
- Identify inappropriate content before making an image visible.
- Normalize recognition results for the Backend and Frontend.

## 2. Work plan

> **Week 5 schedule:** Monday, 20/07/2026 – Friday, 24/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study DetectLabels and DetectModerationLabels | Select the correct API for each purpose |
| Tuesday | Build a Rekognition service in NestJS | Analyze an image from S3 |
| Wednesday | Filter labels by confidence and category | Produce FoodieRecipe-relevant results |
| Thursday | Build approve/reject rules and persist results | Automate content control |
| Friday | Test varied food images and edge cases | Evaluate accuracy and thresholds |

## 3. Work completed

### 3.1. Calling Amazon Rekognition

- Passed the bucket and object key without downloading the image through the Backend.
- Ran label detection and content moderation after upload confirmation.
- Limited the number of labels and set confidence thresholds to reduce noise.
- Handled timeouts, throttling, and missing-object errors.

### 3.2. Result normalization

Converted the response into an internal structure containing label name, confidence, result type, and analysis time. Labels related to food, dishes, or ingredients are prioritized as input for the Bedrock stage.

### 3.3. Image moderation

- Set the image to `failed` with reason `moderation_rejected` when a moderation label exceeded the threshold.
- Used the same `failed` state with reason `moderation_review` for uncertain cases requiring manual review.
- Kept accepted images in `processing` while forwarding them to the Bedrock stage.

## 4. Knowledge and skills gained

- Used Rekognition with S3 objects.
- Understood confidence scores and threshold selection.
- Designed automated moderation with a human-review fallback.
- Normalized AI output for downstream processing.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| Labels were too generic | Filtered by FoodieRecipe categories and confidence |
| Complex dishes produced many results | Kept top labels and let Bedrock interpret their context |
| Moderation results could be incorrect | Used conservative thresholds and a manual-review state |

## 6. Deliverables

- A Rekognition service integrated into NestJS.
- Label detection and image moderation for S3 objects.
- Normalized results for the Frontend and Amazon Bedrock.

## 7. Next-week plan

Use Amazon Bedrock to analyze images and labels and suggest recipe content.
