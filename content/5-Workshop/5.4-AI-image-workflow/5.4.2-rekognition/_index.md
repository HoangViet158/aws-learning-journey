---
title: "Detect and Moderate with Rekognition"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### 1. Install the package

```bash
npm install @aws-sdk/client-rekognition
```

Rekognition reads the image directly from S3 by bucket/key, so the EC2 role needs Rekognition permissions and read access to `uploads/`.

#### 2. Create the Rekognition service

```ts
import {
  DetectLabelsCommand,
  DetectModerationLabelsCommand,
  RekognitionClient,
} from '@aws-sdk/client-rekognition';

const client = new RekognitionClient({ region: process.env.AWS_REGION });

export async function analyzeImage(key: string) {
  const image = { S3Object: { Bucket: process.env.AWS_BUCKET_NAME, Name: key } };

  const moderation = await client.send(new DetectModerationLabelsCommand({
    Image: image,
    MinConfidence: 80,
  }));

  const labels = await client.send(new DetectLabelsCommand({
    Image: image,
    MaxLabels: 20,
    MinConfidence: 70,
  }));

  return {
    moderation: moderation.ModerationLabels ?? [],
    labels: labels.Labels ?? [],
  };
}
```

{{% notice note %}}
The example confidence thresholds are starting values, not absolute standards. Evaluate them with a representative FoodieRecipe image set.
{{% /notice %}}

#### 3. Apply moderation rules

1. Run `DetectModerationLabels` first.
2. If a blocked label exceeds the threshold, set `failed` and `error_code=moderation_rejected`.
3. If a result needs review, still use `failed` with `error_code=moderation_review`.
4. If accepted, keep the image in `processing`.

Do not create additional `rejected` or `review_required` states.

#### 4. Normalize labels

Store only the necessary data:

```json
{
  "labels": [
    { "name": "Food", "confidence": 99.1 },
    { "name": "Soup", "confidence": 94.3 }
  ],
  "moderation": []
}
```

Select top food/ingredient labels as Bedrock context. Never treat labels as perfectly accurate facts.

#### 5. Handle errors

| Error | Handling |
| ----- | -------- |
| Object does not exist | `failed` + `s3_object_missing` |
| Unsupported format | `failed` + `unsupported_image` |
| Throttling/timeout | Limited retries with exponential backoff |
| No useful labels | Continue to Bedrock with the image and empty context |

#### 6. Test

- A valid food image returns confidence-scored labels.
- An inappropriate image becomes `failed`.
- Logs exclude sensitive or unnecessary full responses.
- Only one job runs per image ID at a time.

Reference: [Amazon Rekognition Image API](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/rekognition/).
