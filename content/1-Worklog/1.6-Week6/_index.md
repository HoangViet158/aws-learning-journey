---
title: "Week 6: Image Analysis with Amazon Bedrock"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

## 1. Objectives

- Integrate a multimodal model through Amazon Bedrock.
- Combine the S3 image and Rekognition labels to understand food context.
- Suggest a dish name, description, ingredients, and recipe tags.
- Request structured output and validate it before use.

## 2. Work plan

> **Week 6 schedule:** Monday, 27/07/2026 – Friday, 31/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study Bedrock Runtime and multimodal model capabilities | Select an appropriate image and prompt format |
| Tuesday | Build a Bedrock service in NestJS | Invoke the model and receive a response |
| Wednesday | Design prompts combining images and Rekognition labels | Improve result relevance |
| Thursday | Add JSON schema, validation, and fallback handling | Process model responses reliably |
| Friday | Evaluate quality, latency, token usage, and cost | Establish suitable prompts and usage limits |

## 3. Work completed

### 3.1. Input preparation

- Sent only images that passed Rekognition moderation.
- Normalized image size and format for model limits.
- Included high-confidence labels as additional context.
- Excluded unnecessary personal or sensitive user information from prompts.

### 3.2. Structured prompts and results

Asked the model to return JSON containing a suggested dish name, short description, recognizable ingredients, tags, and confidence. NestJS parses and validates the response; AI output remains a suggestion that the user reviews before saving.

### 3.3. Reliability and cost

- Limited output length and retry count.
- Cached results by image hash to avoid repeated analysis.
- Recorded the model ID, prompt version, and processing time for traceability.
- Clearly labeled AI suggestions and allowed user edits.

## 4. Knowledge and skills gained

- Invoked a multimodal model through Amazon Bedrock Runtime.
- Designed prompts using image and Rekognition context.
- Parsed, validated, and recovered from malformed AI output.
- Balanced result quality, latency, and model cost.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| AI output did not match JSON | Specified a strict schema and validated it in the Backend |
| Results contained uncertain information | Added confidence, used suggestion wording, and required user review |
| Reanalyzing images increased cost | Cached by image hash and reused valid results |

## 6. Deliverables

- A NestJS Bedrock service receiving normalized images and labels.
- A structured prompt for recipe suggestions.
- Validation, caching, fallback, and user-confirmation handling.

## 7. Next-week plan

Place CloudFront in front of S3 to accelerate and control FoodieRecipe image access.
