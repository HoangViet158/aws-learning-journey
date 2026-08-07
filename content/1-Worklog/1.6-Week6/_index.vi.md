---
title: "Tuần 6: Phân tích ảnh với Amazon Bedrock"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

## 1. Mục tiêu

- Tích hợp Amazon Bedrock Runtime vào Backend NestJS.
- Dùng label và nguyên liệu từ Rekognition để xây prompt tạo công thức.
- Sinh tên món, mô tả, nguyên liệu, bước nấu và thông tin dinh dưỡng.
- Parse, validate và lưu kết quả có cấu trúc vào PostgreSQL.

## 2. Kế hoạch công việc

> **Thời gian tuần 6:** Thứ 2, 27/07/2026 – Thứ 6, 31/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu Bedrock Runtime, Converse API và quyền `InvokeModel` | Xác định model và cấu hình gọi dịch vụ |
| Thứ 3 | Xây `BedrockService` trong NestJS | Gọi model bằng prompt văn bản thành công |
| Thứ 4 | Thiết kế prompt từ label/nguyên liệu Rekognition | Nhận công thức tiếng Việt theo JSON schema |
| Thứ 5 | Parse, validate, dịch tên nguyên liệu và lưu recipe/history | Hoàn thiện luồng dữ liệu AI |
| Thứ 6 | Kiểm thử giao diện, lỗi JSON, latency và giới hạn token | Pipeline hoạt động ổn định |

## 3. Nội dung thực hiện

### 3.1. Tạo prompt từ kết quả Rekognition

Trong implementation hiện tại, Bedrock không nhận trực tiếp file ảnh. Ảnh được Rekognition phân tích trước; `PromptBuilderService` nhận danh sách nguyên liệu đã lọc cùng confidence rồi tạo prompt văn bản. Prompt yêu cầu model viết toàn bộ nội dung bằng tiếng Việt, sử dụng chủ yếu nguyên liệu nhận diện được và chỉ trả về JSON hợp lệ.

Các trường đầu ra gồm `detectedIngredients`, `title`, `description`, `cookTime`, `difficulty`, `servings`, `ingredients`, `steps`, `tips` và `nutrition`. `sourceName` được giữ nguyên để Backend ánh xạ tên label tiếng Anh sang tên nguyên liệu tiếng Việt.

### 3.2. Gọi Amazon Bedrock Runtime

`BedrockService` dùng `BedrockRuntimeClient` và `ConverseCommand`. Model được đọc từ `BEDROCK_MODEL_ID`; giới hạn output là 1.500 token và temperature là `0.7`.

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

Môi trường local dùng AWS credential chain đã cấu hình; EC2 dùng IAM Role có `bedrock:InvokeModel` trên model cần thiết. Không đưa access key vào source code hoặc ảnh Docker.

**Frontend hiển thị trạng thái trong khi chờ Bedrock tạo công thức:**

![Amazon Bedrock đang xử lý yêu cầu tạo công thức](/images/1-Worklog/1.6-Week6/bedrock-processing.png)

### 3.3. Parse, validate và lưu công thức

Backend trích JSON từ response Bedrock, gọi `JSON.parse` rồi kiểm tra tối thiểu `title`, `ingredients` và `steps`. Nếu response không có JSON, JSON sai cú pháp hoặc thiếu trường bắt buộc, API trả `BadRequestException` và ghi lịch sử `FAILED`.

Khi dữ liệu hợp lệ, Backend ánh xạ tên nguyên liệu, lưu recipe với nguồn `AI_BEDROCK`, tạo các ingredient/step liên quan trong transaction và ghi `ai_generation_history` với model, prompt, label, recipe ID cùng trạng thái `SUCCESS`.

Response cho Frontend gồm:

```json
{
  "labels": [{ "name": "Bread", "confidence": 98.9 }],
  "ingredients": [{ "name": "Bánh mì", "confidence": 98.9 }],
  "recipe": {
    "id": 1,
    "title": "Tên công thức",
    "description": "Mô tả",
    "cookTime": 15,
    "difficulty": "Dễ",
    "servings": 4,
    "ingredients": [],
    "steps": []
  },
  "historyId": 1
}
```

**Giao diện hiển thị label Rekognition và công thức Bedrock đã tạo:**

![Kết quả tạo công thức từ Amazon Bedrock](/images/1-Worklog/1.6-Week6/bedrock-recipe-result.png)

**Trang chi tiết hiển thị nguyên liệu và các bước thực hiện của công thức AI:**

![Chi tiết công thức được tạo bởi AI](/images/1-Worklog/1.6-Week6/generated-recipe-details.png)

### 3.4. Độ tin cậy và kiểm soát chi phí

- Giới hạn `maxTokens`, chỉ gọi Bedrock sau khi Rekognition trả về dữ liệu phù hợp.
- Ghi model ID, prompt, label, trạng thái và thời điểm tạo để truy vết.
- Không tự động retry không giới hạn khi model trả JSON sai.
- Hiển thị kết quả dưới dạng nội dung do AI tạo để người dùng kiểm tra trước khi sử dụng.

## 4. Kiến thức và kỹ năng đạt được

- Gọi model bằng Bedrock Runtime Converse API.
- Thiết kế prompt chặt chẽ từ dữ liệu Rekognition và JSON schema.
- Parse, validate, bản địa hóa và lưu response trong transaction.
- Theo dõi lịch sử thành công/thất bại và giới hạn chi phí bằng token.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| AI trả dữ liệu không đúng JSON | Yêu cầu chỉ trả JSON, trích object và validate trường bắt buộc |
| Tên nguyên liệu từ Rekognition là tiếng Anh | Yêu cầu `sourceName` cố định và tên dịch tiếng Việt trong prompt |
| Response hợp lệ nhưng lưu dữ liệu thất bại | Dùng transaction và ghi lịch sử `FAILED` |
| Model tạo output dài làm tăng chi phí | Giới hạn `maxTokens` và chỉ gửi các label đã lọc |

## 6. Kết quả đầu ra

- `BedrockService` gọi model qua `ConverseCommand` thành công.
- Prompt tạo công thức tiếng Việt theo JSON schema từ label Rekognition.
- Công thức, nguyên liệu, bước nấu và lịch sử AI được lưu vào PostgreSQL.
- Frontend hiển thị trạng thái xử lý, label, công thức tóm tắt và trang chi tiết.

## 7. Kế hoạch tuần tiếp theo

Cấu hình CloudFront trước S3 để tăng tốc và kiểm soát truy cập ảnh FoodieRecipe.
