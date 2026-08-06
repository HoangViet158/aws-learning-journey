---
title: "Tuần 8: Tích hợp và hoàn thiện pipeline ảnh"
date: 2026-08-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

## 1. Mục tiêu

- Hoàn thiện pipeline Next.js–NestJS–S3–Rekognition–Bedrock–CloudFront.
- Kiểm thử chức năng, bảo mật, hiệu năng và các trường hợp lỗi.
- Rà soát quyền IAM và chi phí của từng dịch vụ AWS.
- Hoàn thiện sơ đồ kiến trúc, tài liệu API và báo cáo công việc.

## 2. Kế hoạch công việc

> **Thời gian tuần 8:** Thứ 2, 10/08/2026 – Thứ 6, 14/08/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Kiểm thử end-to-end từ chọn ảnh đến hiển thị kết quả AI | Xác nhận luồng chính hoạt động đúng |
| Thứ 3 | Kiểm thử file lỗi, timeout, retry và AI output không hợp lệ | Pipeline có khả năng phục hồi |
| Thứ 4 | Rà IAM, S3 policy, OAC, CORS và dữ liệu nhạy cảm | Thu hẹp quyền và rủi ro bảo mật |
| Thứ 5 | Đo latency, cache hit và chi phí Rekognition/Bedrock | Xác định điểm cần tối ưu |
| Thứ 6 | Hoàn thiện tài liệu, sơ đồ, demo và báo cáo | Sẵn sàng bàn giao phần việc |

## 3. Nội dung thực hiện

### 3.1. Pipeline hoàn chỉnh

1. Next.js yêu cầu NestJS tạo image record và pre-signed URL.
2. Trình duyệt upload ảnh trực tiếp lên S3 rồi xác nhận với NestJS.
3. NestJS gọi Rekognition để nhận diện label và kiểm duyệt nội dung.
4. Ảnh hợp lệ cùng label được gửi đến Bedrock để gợi ý thông tin công thức.
5. Kết quả được người dùng kiểm tra; ảnh được hiển thị qua CloudFront.

### 3.2. Kiểm thử và xử lý lỗi

- Kiểm tra file sai định dạng, quá dung lượng, upload bị ngắt và URL hết hạn.
- Kiểm tra ảnh bị Rekognition từ chối hoặc yêu cầu review.
- Giả lập Bedrock timeout, output sai schema và retry vượt giới hạn.
- Xác nhận ảnh cũ được dọn và trạng thái không bị kẹt giữa pipeline.

### 3.3. Bảo mật và chi phí

- Chỉ cấp quyền S3, Rekognition và Bedrock cần thiết cho Backend.
- Giữ S3 private; người dùng đọc ảnh qua CloudFront.
- Không log AWS credential, pre-signed URL đầy đủ hoặc dữ liệu nhạy cảm.
- Cache kết quả AI, giới hạn retry và theo dõi số lần gọi dịch vụ.

### 3.4. Tài liệu bàn giao

Hoàn thiện sequence diagram, mô tả trạng thái ảnh, API contract, biến môi trường, bảng lỗi, quy trình kiểm thử và hướng dẫn xử lý sự cố. Chuẩn bị demo từ upload ảnh đến kết quả nhận diện và URL CloudFront.

## 4. Kiến thức và kỹ năng đạt được

- Xây dựng luồng ảnh xuyên suốt giữa Next.js và NestJS.
- Tích hợp S3, Rekognition, Bedrock và CloudFront theo từng trách nhiệm rõ ràng.
- Thiết kế AI workflow có validation, fallback và human review.
- Đánh giá bảo mật, hiệu năng và chi phí của pipeline xử lý ảnh.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| Trạng thái bất đồng bộ giữa các bước | Dùng image ID chung và state machine rõ ràng |
| Kết quả AI khó tái hiện | Lưu model ID, prompt version, input hash và response đã chuẩn hóa |
| Khó xác định bước gây chậm | Ghi thời gian từng bước upload, Rekognition, Bedrock và CloudFront |

## 6. Kết quả tổng kết

- Hoàn thành tính năng upload và quản lý ảnh bằng Next.js, NestJS và S3.
- Rekognition thực hiện nhận diện/kiểm duyệt; Bedrock gợi ý nội dung công thức.
- CloudFront phân phối ảnh từ S3 với cache để cải thiện tốc độ truy cập.
- Pipeline có validation, xử lý lỗi, kiểm soát quyền và tài liệu đầy đủ.
