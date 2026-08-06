---
title: "Workshop"
date: 2026-06-22
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng workflow xử lý ảnh AI cho FoodieRecipe trên AWS

#### Tổng quan

Workshop hướng dẫn xây dựng luồng xử lý ảnh cho ứng dụng chia sẻ công thức **FoodieRecipe**. Next.js gửi yêu cầu đến NestJS, upload ảnh trực tiếp lên Amazon S3 bằng pre-signed URL, dùng Amazon Rekognition để nhận diện/kiểm duyệt, dùng Amazon Bedrock để gợi ý nội dung công thức và dùng Amazon CloudFront để phân phối ảnh nhanh hơn.

Kiến trúc sản phẩm hoàn chỉnh sử dụng NestJS chạy trong Docker trên EC2 sau Nginx, Amazon RDS lưu dữ liệu, AWS Secrets Manager quản lý bí mật, IAM cấp quyền và Amazon CloudWatch thu thập log/metrics.

{{% notice note %}}
Workshop sử dụng bốn trạng thái ảnh: `pending`, `processing`, `completed` và `failed`.
{{% /notice %}}

#### Nội dung

1. [Tổng quan Workshop](5.1-workshop-overview/)
2. [Chuẩn bị môi trường](5.2-prerequisites/)
3. [Xây dựng hạ tầng cốt lõi](5.3-core-infrastructure/)
4. [Xây dựng workflow xử lý ảnh AI](5.4-ai-image-workflow/)
5. [Kiểm thử, bảo mật và giám sát](5.5-validation-monitoring/)
6. [Dọn dẹp tài nguyên](5.6-cleanup/)

#### Kết quả sau Workshop

- Tạo được S3 image bucket với hai prefix `uploads/` và `delivery/`.
- Xây dựng được API NestJS cấp pre-signed URL và quản lý trạng thái ảnh.
- Tích hợp được Rekognition và Bedrock.
- Cấu hình được CloudFront với S3 private origin.
- Kiểm tra được log, lỗi, quyền IAM và toàn bộ luồng end-to-end.
