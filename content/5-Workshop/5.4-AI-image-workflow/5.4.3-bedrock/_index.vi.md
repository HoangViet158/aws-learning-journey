---
title: "Gợi ý công thức với Amazon Bedrock"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### 1. Chọn model

1. Mở Amazon Bedrock Console trong Region đã chọn.
2. Xác nhận tài khoản được phép sử dụng một model đa phương thức hỗ trợ ảnh.
3. Lưu model ID vào `BEDROCK_MODEL_ID`.
4. Kiểm tra EC2 role có quyền `bedrock:InvokeModel` phù hợp.

{{% notice warning %}}
Khả năng, model ID và Region hỗ trợ có thể thay đổi. Không hard-code model ID vào source code; dùng biến môi trường hoặc cấu hình được quản lý.
{{% /notice %}}

#### 2. Cài package

```bash
npm install @aws-sdk/client-bedrock-runtime
```

#### 3. Chuẩn bị prompt

Prompt cần yêu cầu output JSON rõ ràng:

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

#### 4. Gọi Bedrock Runtime

Đọc object trong prefix `uploads/` thành bytes, sau đó gửi ảnh và prompt đến model:

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

Giá trị `format` phải khớp ảnh thực tế. Chuẩn hóa JPEG/PNG/WebP trước khi gọi model nếu cần.

#### 5. Validate kết quả

1. Loại bỏ code fence nếu model trả về.
2. Parse JSON trong `try/catch`.
3. Validate schema bằng thư viện của dự án.
4. Giới hạn số phần tử và độ dài chuỗi.
5. Nếu output sai sau số lần retry cho phép, đặt `failed` với `bedrock_invalid_output`.
6. Lưu model ID, prompt version, latency và kết quả đã chuẩn hóa.

{{% notice note %}}
AI chỉ gợi ý nội dung. Người dùng phải xem lại tên món, mô tả và nguyên liệu trước khi lưu công thức.
{{% /notice %}}

#### 6. Cache và kiểm soát chi phí

- Tạo checksum/hash từ ảnh.
- Tái sử dụng kết quả khi cùng hash, model ID và prompt version.
- Giới hạn output token, retry và số request đồng thời.
- Không gọi Bedrock nếu Rekognition moderation thất bại.

#### 7. Kiểm tra

- Output hợp lệ đúng JSON schema.
- Output có markdown/code fence vẫn được parse an toàn.
- Timeout hoặc throttling được retry có giới hạn.
- Không hiển thị kết quả AI như dữ liệu đã được xác nhận.

Tài liệu tham khảo: [Amazon Bedrock Runtime Converse API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html).
