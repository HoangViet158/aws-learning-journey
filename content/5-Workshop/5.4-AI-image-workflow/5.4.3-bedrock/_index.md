---
title: "Suggest Recipe Content with Amazon Bedrock"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### 1. Select a model

1. Open the Amazon Bedrock Console in the selected Region.
2. Confirm that the account can use a multimodal model supporting images.
3. Store the model ID in `BEDROCK_MODEL_ID`.
4. Confirm that the EC2 role has the applicable `bedrock:InvokeModel` permission.

{{% notice warning %}}
Capabilities, model IDs, and Region support can change. Do not hard-code the model ID in source code; use an environment variable or managed configuration.
{{% /notice %}}

#### 2. Install the package

```bash
npm install @aws-sdk/client-bedrock-runtime
```

#### 3. Prepare the prompt

Require a clear JSON output:

```text
Analyze this food image and the Rekognition labels below.
Return JSON only with this schema:
{
  "dishName": "string",
  "description": "string",
  "ingredients": ["string"],
  "tags": ["string"],
  "confidence": 0
}
Treat every field as a suggestion. Do not invent quantities.
Labels: <normalized-labels>
```

#### 4. Invoke Bedrock Runtime

Read the object under `uploads/` into bytes, then send the image and prompt to the model:

```ts
import { GetObjectCommand, S3Client } from '@aws-sdk/client-s3';
import { BedrockRuntimeClient, ConverseCommand } from '@aws-sdk/client-bedrock-runtime';

const s3 = new S3Client({ region: process.env.AWS_REGION });
const bedrock = new BedrockRuntimeClient({ region: process.env.AWS_REGION });

async function suggestRecipe(key: string, labels: unknown[]) {
  const object = await s3.send(new GetObjectCommand({
    Bucket: process.env.AWS_BUCKET_NAME,
    Key: key,
  }));
  const bytes = await object.Body!.transformToByteArray();

  const result = await bedrock.send(new ConverseCommand({
    modelId: process.env.BEDROCK_MODEL_ID,
    messages: [{
      role: 'user',
      content: [
        { image: { format: 'jpeg', source: { bytes } } },
        { text: buildPrompt(labels) },
      ],
    }],
    inferenceConfig: { maxTokens: 800, temperature: 0.2 },
  }));

  return result.output?.message?.content?.find(item => item.text)?.text;
}
```

The `format` value must match the actual image. Normalize JPEG/PNG/WebP before invocation when necessary.

#### 5. Validate the result

1. Remove code fences if the model returned them.
2. Parse JSON inside `try/catch`.
3. Validate the schema with the project's validation library.
4. Limit array counts and string lengths.
5. If output remains invalid after allowed retries, set `failed` with `bedrock_invalid_output`.
6. Store the model ID, prompt version, latency, and normalized result.

{{% notice note %}}
AI output is only a suggestion. The user must review dish names, descriptions, and ingredients before saving the recipe.
{{% /notice %}}

#### 6. Cache and control cost

- Create a checksum/hash for each image.
- Reuse a result when the hash, model ID, and prompt version match.
- Limit output tokens, retries, and concurrent requests.
- Never invoke Bedrock after failed Rekognition moderation.

#### 7. Test

- Valid output matches the JSON schema.
- Output containing markdown/code fences is parsed safely.
- Timeout or throttling uses limited retries.
- AI output is never presented as confirmed fact.

Reference: [Amazon Bedrock Runtime Converse API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html).
