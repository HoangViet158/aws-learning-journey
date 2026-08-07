---
title: "Week 6: Image Analysis with Amazon Bedrock"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

## 1. Objectives

- Integrate Amazon Bedrock Runtime with the NestJS Backend.
- Build recipe prompts from Rekognition labels and extracted ingredients.
- Generate a title, description, ingredients, steps, and nutrition data.
- Parse, validate, and persist structured results in PostgreSQL.

## 2. Work plan

> **Week 6 schedule:** Monday, 27/07/2026 – Friday, 31/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study Bedrock Runtime, the Converse API, and `InvokeModel` permission | Define the model and service configuration |
| Tuesday | Build `BedrockService` in NestJS | Invoke the model with a text prompt successfully |
| Wednesday | Build a prompt from Rekognition labels/ingredients | Receive a Vietnamese recipe matching the JSON schema |
| Thursday | Parse, validate, localize ingredient names, and persist recipe/history | Complete the AI data flow |
| Friday | Test the interface, invalid JSON, latency, and token limits | Stabilize the pipeline |

## 3. Work completed

### 3.1. Building a prompt from Rekognition results

In the current implementation, Bedrock does not receive the image file directly. Rekognition analyzes the image first; `PromptBuilderService` receives the filtered ingredients with confidence values and builds a text prompt. The prompt requires the model to write entirely in Vietnamese, rely primarily on detected ingredients, and return valid JSON only.

The output fields are `detectedIngredients`, `title`, `description`, `cookTime`, `difficulty`, `servings`, `ingredients`, `steps`, `tips`, and `nutrition`. Each original `sourceName` remains unchanged so the Backend can map the English label to its Vietnamese ingredient name.

### 3.2. Invoking Amazon Bedrock Runtime

`BedrockService` uses `BedrockRuntimeClient` and `ConverseCommand`. The model comes from `BEDROCK_MODEL_ID`; output is limited to 1,500 tokens with a temperature of `0.7`.

```ts
const command = new ConverseCommand({
  modelId: config.getOrThrow<string>('BEDROCK_MODEL_ID'),
  messages: [
    {
      role: ConversationRole.USER,
      content: [{ text: prompt }],
    },
  ],
  inferenceConfig: {
    maxTokens: 1500,
    temperature: 0.7,
  },
});

const response = await bedrockClient.send(command);
```

```env
AWS_REGION=
BEDROCK_MODEL_ID=
```

Local development uses the configured AWS credential chain; EC2 uses an IAM role with `bedrock:InvokeModel` for the required model. Access keys are never embedded in source code or the Docker image.

**The Frontend displays a processing state while Bedrock generates the recipe:**

![Amazon Bedrock processing a recipe-generation request](/images/1-Worklog/1.6-Week6/bedrock-processing.png)

### 3.3. Parsing, validating, and persisting the recipe

The Backend extracts JSON from the Bedrock response, calls `JSON.parse`, and validates at least `title`, `ingredients`, and `steps`. If the response contains no JSON, malformed JSON, or missing required fields, the API returns `BadRequestException` and records a `FAILED` history entry.

For valid data, the Backend maps ingredient names, saves the recipe with source `AI_BEDROCK`, creates related ingredients and steps in a transaction, and stores `ai_generation_history` with the model, prompt, labels, recipe ID, and `SUCCESS` status.

The Frontend response includes:

```json
{
  "labels": [{ "name": "Bread", "confidence": 98.9 }],
  "ingredients": [{ "name": "Bánh mì", "confidence": 98.9 }],
  "recipe": {
    "id": 1,
    "title": "Recipe title",
    "description": "Description",
    "cookTime": 15,
    "difficulty": "Easy",
    "servings": 4,
    "ingredients": [],
    "steps": []
  },
  "historyId": 1
}
```

**The interface displays Rekognition labels and the Bedrock-generated recipe:**

![Recipe-generation result from Amazon Bedrock](/images/1-Worklog/1.6-Week6/bedrock-recipe-result.png)

**The detail page presents ingredients and steps from the AI-generated recipe:**

![AI-generated recipe details](/images/1-Worklog/1.6-Week6/generated-recipe-details.png)

### 3.4. Reliability and cost control

- Limit `maxTokens` and invoke Bedrock only after Rekognition produces useful input.
- Record the model ID, prompt, labels, status, and creation time for traceability.
- Avoid unlimited automatic retries when the model returns invalid JSON.
- Clearly present the result as AI-generated content for user review.

## 4. Knowledge and skills gained

- Invoked a model through the Bedrock Runtime Converse API.
- Designed a strict prompt from Rekognition data and a JSON schema.
- Parsed, validated, localized, and persisted responses in a transaction.
- Tracked successful/failed history and controlled token-related cost.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| AI output did not match JSON | Required JSON-only output, extracted the object, and validated required fields |
| Rekognition ingredient names were in English | Preserved `sourceName` and requested Vietnamese names in the prompt |
| The response was valid but persistence failed | Used a transaction and recorded `FAILED` history |
| Long model output increased cost | Limited `maxTokens` and sent only filtered labels |

## 6. Deliverables

- `BedrockService` successfully invoking the model through `ConverseCommand`.
- A Vietnamese recipe prompt following a JSON schema built from Rekognition labels.
- Recipes, ingredients, steps, and AI history persisted in PostgreSQL.
- Frontend processing, label, recipe-summary, and detail views working end to end.

## 7. Next-week plan

Place CloudFront in front of S3 to accelerate and control FoodieRecipe image access.

## 8. References

- [Using the Amazon Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html)
- [Amazon Bedrock Runtime Converse API Reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)
- [Foundation model inference parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html)
- [IAM access control for Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/security-iam.html)
- [Monitoring Amazon Bedrock with Amazon CloudWatch](https://docs.aws.amazon.com/bedrock/latest/userguide/monitoring.html)
