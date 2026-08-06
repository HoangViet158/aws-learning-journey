---
title: "Validation, Security, and Monitoring"
date: 2026-06-22
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

This section verifies correct workflow behavior, traceable failures, and appropriately scoped permissions.

#### 1. End-to-end tests

| Scenario | Expected result |
| -------- | --------------- |
| Valid JPEG/PNG/WebP upload | `pending → processing → completed` |
| Invalid/oversized file | Rejected before signing or moved to `failed` |
| Expired pre-signed URL | S3 rejects it; the user requests a new URL |
| Moderation violation | `failed` with a moderation reason code |
| Rekognition timeout | Limited retry without duplicate jobs |
| Bedrock returns invalid JSON | Validation failure with controlled retry/fallback |
| Copy from `uploads/` to `delivery/` fails | State never becomes `completed` |
| Direct S3 URL | 403 |
| CloudFront URL | 200 with caching behavior |

Use a separate image ID for each test and record request ID, latency, and final state.

#### 2. Structured logging

NestJS should write JSON logs containing:

```json
{
  "event": "image.processing.completed",
  "imageId": "uuid",
  "userId": "masked-or-internal-id",
  "stage": "bedrock",
  "durationMs": 820,
  "status": "completed"
}
```

Never log passwords, tokens, AWS credentials, raw secrets, complete pre-signed URLs, or full image/base64 content.

#### 3. Configure CloudWatch Logs

1. Create `/foodierecipe/api`.
2. Set a workshop-appropriate retention period.
3. Configure CloudWatch Agent or Docker logging for NestJS/Nginx logs.
4. Confirm the EC2 role can create log streams and write log events.
5. Search by `imageId` to trace the complete workflow.

Example Logs Insights query:

```text
fields @timestamp, event, imageId, stage, durationMs, status
| filter imageId = "<image-id>"
| sort @timestamp asc
```

#### 4. Metrics and alarms

Track at minimum:

- Counts of `completed` and `failed` images.
- Upload, Rekognition, Bedrock, and S3-copy latency.
- Error count by `error_code`.
- Retry and Bedrock-invocation counts.
- CloudFront requests, error rate, and cache-hit ratio.
- EC2 CPU/disk and RDS connections/storage.

Create alarms for elevated failure rate, constrained EC2 resources, CloudFront 5xx errors, and budget thresholds.

#### 5. Security review

1. The S3 image bucket retains Block Public Access.
2. CloudFront can read only `delivery/` through OAC.
3. `uploads/` accepts signed PUT requests only from allowed CORS origins.
4. EC2 uses an IAM role without static access keys.
5. RDS is private and accepts only Backend connections.
6. Secrets are absent from Git, Docker images, and logs.
7. IAM avoids `s3:*`, `rekognition:*`, and `bedrock:*` when unnecessary.
8. APIs check ownership before image confirmation/deletion.

#### 6. Workshop completion criteria

- A valid image completes the flow and displays through a CloudFront URL.
- The AI suggestion matches the schema and is editable.
- A moderated image becomes `failed` with a reason.
- The direct S3 URL is inaccessible.
- One image ID can be traced through CloudWatch Logs.
- No public permission or hard-coded credential remains.

After validation, continue to [Clean up resources](../5.6-cleanup/).
