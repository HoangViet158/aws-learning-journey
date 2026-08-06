---
title: "Nhận diện và kiểm duyệt với Rekognition"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### 1. Cài package

```bash
npm install @aws-sdk/client-rekognition
```

Rekognition đọc ảnh trực tiếp từ S3 bằng bucket/key, vì vậy EC2 role cần quyền Rekognition và quyền đọc prefix `uploads/`.

#### 2. Tạo Rekognition service

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
Ngưỡng confidence trong ví dụ là giá trị khởi đầu, không phải tiêu chuẩn tuyệt đối. Hãy đánh giá bằng bộ ảnh FoodieRecipe thực tế.
{{% /notice %}}

#### 3. Áp dụng quy tắc kiểm duyệt

1. Chạy `DetectModerationLabels` trước.
2. Nếu có label bị chặn vượt ngưỡng, đặt trạng thái `failed` và `error_code=moderation_rejected`.
3. Nếu kết quả cần xem lại, vẫn dùng `failed` với `error_code=moderation_review`.
4. Nếu ảnh đạt yêu cầu, tiếp tục trạng thái `processing`.

Không tạo thêm các trạng thái `rejected` hoặc `review_required`.

#### 4. Chuẩn hóa label

Chỉ lưu dữ liệu cần thiết:

```json
{
  "labels": [
    { "name": "Food", "confidence": 99.1 },
    { "name": "Soup", "confidence": 94.3 }
  ],
  "moderation": []
}
```

Lọc top label liên quan đến món ăn/nguyên liệu để làm ngữ cảnh cho Bedrock. Không coi label là dữ liệu chính xác tuyệt đối.

#### 5. Xử lý lỗi

| Lỗi | Xử lý |
| --- | ----- |
| Object không tồn tại | `failed` + `s3_object_missing` |
| Định dạng không được hỗ trợ | `failed` + `unsupported_image` |
| Throttling/timeout | Retry có exponential backoff và giới hạn số lần |
| Không có label hữu ích | Tiếp tục Bedrock với ảnh và context rỗng |

#### 6. Kiểm tra

- Ảnh món ăn hợp lệ trả về label có confidence.
- Ảnh không phù hợp bị đánh dấu `failed`.
- Không log nội dung nhạy cảm hoặc toàn bộ response không cần thiết.
- Mỗi image ID chỉ chạy một job tại một thời điểm.

Tài liệu tham khảo: [Amazon Rekognition Image API](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/rekognition/).
