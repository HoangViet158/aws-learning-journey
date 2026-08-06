---
title: "Tuần 1: Nền tảng AWS và thiết kế FoodieRecipe"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## 1. Mục tiêu

- Hiểu các khái niệm cơ bản của điện toán đám mây và AWS.
- Thiết lập tài khoản AWS an toàn bằng IAM và MFA.
- Cấu hình AWS Budget Alert để kiểm soát chi phí.
- Phân tích yêu cầu và thiết kế pipeline xử lý ảnh cho FoodieRecipe.

## 2. Kế hoạch công việc

> **Thời gian tuần 1:** Thứ 2, 22/06/2026 – Thứ 6, 26/06/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu Cloud Computing, Region, Availability Zone và các dịch vụ AWS chính | Nắm được nền tảng AWS |
| Thứ 3 | Tạo tài khoản, bật MFA và tìm hiểu mô hình Shared Responsibility | Tài khoản được bảo vệ cơ bản |
| Thứ 4 | Tạo IAM user/group, áp dụng quyền tối thiểu và cấu hình AWS CLI | Truy cập AWS không dùng root user |
| Thứ 5 | Tạo Budget Alert và bật thông báo chi phí | Có cơ chế cảnh báo ngân sách |
| Thứ 6 | Phân tích chức năng và kiến trúc xử lý ảnh FoodieRecipe | Có phạm vi và sơ đồ pipeline ban đầu |

## 3. Nội dung thực hiện

### 3.1. Thiết lập tài khoản và bảo mật IAM

- Bật MFA cho root user và hạn chế sử dụng tài khoản root.
- Tạo IAM group dành cho phát triển, gán policy theo nguyên tắc **least privilege**.
- Tạo IAM user để thao tác trên Console và AWS CLI.
- Kiểm tra thông tin xác thực bằng lệnh `aws sts get-caller-identity`.

### 3.2. Quản lý ngân sách

- Tạo AWS Budget theo hạn mức chi phí hàng tháng.
- Cấu hình ngưỡng cảnh báo khi chi phí thực tế hoặc dự báo đạt mức quy định.
- Đăng ký địa chỉ email nhận thông báo và kiểm tra trạng thái Budget.

### 3.3. Phân tích dự án FoodieRecipe

Xác định các chức năng chính liên quan đến ảnh: chọn và xem trước ảnh, upload, kiểm tra nội dung, nhận diện món ăn, gợi ý thông tin công thức và hiển thị ảnh với tốc độ ổn định. Kiến trúc sử dụng Next.js cho Frontend, NestJS cho Backend, Amazon S3 để lưu ảnh, Amazon Rekognition và Amazon Bedrock để phân tích AI, Amazon CloudFront để phân phối ảnh.

## 4. Kiến thức và kỹ năng đạt được

- Phân biệt IaaS, PaaS, SaaS và vai trò của các dịch vụ AWS trong dự án.
- Hiểu vai trò của IAM user, group, role và policy.
- Biết bảo vệ tài khoản bằng MFA và nguyên tắc quyền tối thiểu.
- Biết lập ngân sách, cảnh báo chi phí và xác định phạm vi dự án.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| Dễ nhầm giữa user, role và policy | Vẽ sơ đồ quan hệ và thử nghiệm với quyền chỉ đọc trước |
| Chưa ước lượng được chi phí | Dùng AWS Pricing Calculator và đặt Budget ở mức thấp |
| Phạm vi dự án ban đầu quá rộng | Ưu tiên chức năng cốt lõi theo mô hình MVP |

## 6. Kết quả đầu ra

- Tài khoản AWS đã bật MFA và có IAM user phục vụ phát triển.
- Budget Alert đã được cấu hình.
- Danh sách yêu cầu và pipeline xử lý ảnh FoodieRecipe đã được xác định.

## 7. Kế hoạch tuần tiếp theo

Tìm hiểu Amazon S3 và thiết kế cơ chế upload, lưu trữ, phân quyền cho ảnh công thức.
