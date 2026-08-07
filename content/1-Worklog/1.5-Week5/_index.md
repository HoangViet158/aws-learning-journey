---
title: "Week 5: Image Recognition with Amazon Rekognition"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## 1. Objectives

- Integrate Amazon Rekognition with images stored in S3.
- Invoke `DetectLabels` from the NestJS Backend to recognize image content.
- Filter and normalize labels by confidence to create an ingredient input list.
- Forward Rekognition results to the Amazon Bedrock recipe-generation stage.

## 2. Work plan

> **Week 5 schedule:** Monday, 20/07/2026 – Friday, 24/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study Rekognition `DetectLabels`, IAM permissions, and the S3 object format | Define the required configuration |
| Tuesday | Build `RekognitionService` and integrate the image-analysis endpoint | Analyze an S3 image from the Backend |
| Wednesday | Filter labels by confidence, remove generic labels, and normalize names | Produce a relevant ingredient list |
| Thursday | Connect Rekognition to the prompt builder and AI history | Complete the Bedrock input flow |
| Friday | Test varied images and AWS SDK failures | Establish thresholds and fallback cases |

## 3. Work completed

### 3.1. Uploading an image and invoking Amazon Rekognition

The Frontend sends the image as `multipart/form-data` to `POST /api/ai/analyze-image`. The endpoint is protected by `AuthGuard`; the Backend obtains the `userId`, uploads the file under the S3 `ai-images/` prefix, and passes the object key to `RekognitionService`. Rekognition reads the S3 object directly, so the Backend does not download the image a second time.

```ts
const command = new DetectLabelsCommand({
  Image: {
    S3Object: {
      Bucket: config.getOrThrow<string>('AWS_BUCKET_NAME'),
      Name: imageKey,
    },
  },
  MaxLabels: 30,
  MinConfidence: 50,
});

return rekognitionClient.send(command);
```

**The user selects an ingredient image before analysis:**

![Ingredient image selected for analysis](/images/1-Worklog/1.5-Week5/ai-image-selected.png)

### 3.2. Filtering and normalizing results

The Backend retains Rekognition results as `{ name, confidence }` and rounds confidence to two decimal places. `IngredientService` accepts labels at **80%** or higher, removes generic labels such as `Food`, `Meal`, `Dish`, `Plate`, and `Ingredient`, normalizes selected synonyms, removes duplicates, and sorts by descending confidence.

Raw labels are still returned to the Frontend for display, while the filtered ingredient list becomes input to `PromptBuilderService` for the Bedrock stage.

**Detected labels and confidence scores are displayed in the interface:**

![Amazon Rekognition food-image detection result](/images/1-Worklog/1.5-Week5/rekognition-labels-result.png)

### 3.3. Persisting results and handling errors

- Log the number of labels returned by Rekognition and the number of extracted ingredients.
- Store normalized labels in `ai_generation_history` together with the user, model, status, and generated recipe.
- If the S3 upload or Rekognition invocation fails, log the error and return a normalized exception without calling Bedrock.
- Distinguish missing objects, missing `rekognition:DetectLabels` permission, Region mismatches, and temporary service failures.

**Detected labels are persisted in AI-generation history:**

![Rekognition labels stored in the database](/images/1-Worklog/1.5-Week5/rekognition-labels-history.png)

### 3.4. Required configuration and permissions

```env
AWS_REGION=
AWS_BUCKET_NAME=
```

During local development, the AWS SDK uses the configured credential chain. On EC2, the application uses an IAM role instead of static access keys. The Backend role requires only `s3:PutObject`, `s3:GetObject` on the image prefix and `rekognition:DetectLabels`.

## 4. Knowledge and skills gained

- Invoked Rekognition with an S3 bucket and object key.
- Understood the difference between the 50% request threshold and the 80% business filter.
- Normalized, deduplicated, and sorted labels before prompt construction.
- Added logging and failure handling around the AWS SDK pipeline.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| Labels were too generic | Applied an ignore list and an 80% business threshold |
| Labels were duplicated or synonymous | Normalized names, removed duplicates, and sorted by confidence |
| Rekognition could not read the object | Verified Region, bucket, object key, and IAM permissions |
| An image produced no useful ingredient | Returned a clear message and asked the user to select another image |

## 6. Deliverables

- `RekognitionService` integrated with NestJS `AIGenerationService`.
- An authenticated endpoint that uploads to S3 and invokes `DetectLabels` successfully.
- Labels filtered, normalized, displayed on the Frontend, and stored in AI history.
- An ingredient list ready for Amazon Bedrock prompt construction.

## 7. Next-week plan

Use Amazon Bedrock to process Rekognition labels/ingredients and generate recipe content.

## 8. References

- [Amazon Rekognition DetectLabels API](https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectLabels.html)
- [Detecting labels in an image with Amazon Rekognition](https://docs.aws.amazon.com/rekognition/latest/dg/labels-detect-labels-image.html)
- [Analyzing images stored in Amazon S3](https://docs.aws.amazon.com/rekognition/latest/dg/images-s3.html)
- [IAM security guidance for Amazon Rekognition](https://docs.aws.amazon.com/rekognition/latest/dg/security-iam.html)
- [Amazon Rekognition service quotas](https://docs.aws.amazon.com/rekognition/latest/dg/limits.html)
