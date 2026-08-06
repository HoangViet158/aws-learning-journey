---
title: "Tổng quan Workshop"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu FoodieRecipe

FoodieRecipe là ứng dụng chia sẻ công thức sử dụng **Next.js** cho Frontend và **NestJS** cho Backend. Người dùng có thể tạo tài khoản, đăng và quản lý công thức, khai báo nguyên liệu/danh mục, tìm kiếm công thức và upload ảnh món ăn.

AI hỗ trợ người dùng bằng cách:

- Kiểm duyệt ảnh và nhận diện món ăn/nguyên liệu với Amazon Rekognition.
- Gợi ý tên món, mô tả, nguyên liệu và tag với Amazon Bedrock.
- Yêu cầu người dùng kiểm tra và chỉnh sửa trước khi lưu.

#### Kiến trúc Workshop

![Kiến trúc FoodieRecipe](/images/2-Proposal/foodie-recipe-architecture.png)

| Thành phần | Vai trò |
| ---------- | ------- |
| Next.js | Giao diện upload, preview, progress và kết quả AI |
| NestJS | API, validation, trạng thái ảnh và điều phối AWS services |
| S3 image bucket | Prefix `uploads/` lưu ảnh gốc; `delivery/` lưu ảnh hoàn tất |
| Rekognition | Nhận diện label và kiểm duyệt nội dung |
| Bedrock | Phân tích ngữ cảnh và gợi ý nội dung công thức |
| CloudFront | Cache và phân phối ảnh từ private S3 origin |
| EC2, Docker, Nginx | Môi trường chạy Backend trong kiến trúc sản phẩm |
| RDS | Lưu dữ liệu nghiệp vụ và trạng thái ảnh |
| Secrets Manager, IAM | Quản lý bí mật và quyền truy cập |
| CloudWatch | Thu thập log, metrics và cảnh báo |

#### Luồng end-to-end

1. Next.js yêu cầu NestJS tạo image record và pre-signed URL.
2. Trình duyệt upload ảnh trực tiếp vào prefix `uploads/` của S3 image bucket.
3. NestJS xác minh object và đặt trạng thái `processing`.
4. Rekognition nhận diện label và kiểm duyệt ảnh.
5. Bedrock tạo gợi ý công thức có cấu trúc.
6. Ảnh hợp lệ được chuyển sang prefix `delivery/` trong cùng bucket.
7. Trạng thái chuyển thành `completed`; Next.js hiển thị ảnh qua CloudFront.
8. Khi xảy ra lỗi, trạng thái là `failed` kèm mã lỗi để retry hoặc kiểm tra.

#### Mục tiêu học tập

- Hiểu cách tách upload khỏi Backend bằng pre-signed URL.
- Thiết kế workflow AI có validation và fallback.
- Bảo vệ S3 bằng Block Public Access và Origin Access Control.
- Áp dụng IAM least privilege và Secrets Manager.
- Theo dõi pipeline bằng CloudWatch.
