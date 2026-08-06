---
title: "Dọn dẹp tài nguyên"
date: 2026-06-22
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Tổng kết

Bạn đã hoàn thành workflow FoodieRecipe từ upload ảnh bằng Next.js đến xử lý qua NestJS, S3, Rekognition, Bedrock và phân phối bằng CloudFront. Phần cuối dọn các tài nguyên workshop để tránh tiếp tục phát sinh chi phí.

{{% notice danger %}}
Chỉ xóa tài nguyên được tạo riêng cho workshop. Không xóa bucket, database, role, secret hoặc log group đang được môi trường khác sử dụng. Sao lưu dữ liệu cần giữ trước khi tiếp tục.
{{% /notice %}}

#### 1. Ngừng phát sinh request

1. Dừng ứng dụng `web` và script đang upload ảnh.
2. Dừng container NestJS hoặc đặt API ở chế độ bảo trì.
3. Xác nhận không còn image record ở trạng thái `processing`.

#### 2. Xóa CloudFront distribution

1. Mở CloudFront và chọn distribution FoodieRecipe.
2. Chọn **Disable** và đợi trạng thái cập nhật hoàn tất.
3. Chọn **Delete**.
4. Xóa Origin Access Control nếu không còn distribution nào dùng.

#### 3. Làm rỗng và xóa S3 image bucket

1. Xóa object trong hai prefix `uploads/` và `delivery/`.
2. Nếu bật versioning, xóa cả object versions và delete markers.
3. Xóa lifecycle rule/CORS nếu cần.
4. Xóa image bucket sau khi bucket trống.

#### 4. Xóa tài nguyên Backend

Nếu đã tạo riêng cho workshop:

1. Dừng và terminate EC2 instance.
2. Xóa Security Group không còn được tham chiếu.
3. Xóa RDS; chỉ tạo final snapshot khi cần giữ dữ liệu.
4. Xóa snapshot/backup thử nghiệm không cần thiết.

#### 5. Xóa bí mật, role và log

1. Schedule deletion cho `foodierecipe/database` nếu secret chỉ dùng cho workshop.
2. Detach policy và xóa `FoodieRecipeBackendRole` sau khi EC2 đã bị xóa.
3. Xóa custom policy không còn được sử dụng.
4. Xóa `/foodierecipe/api` và các alarm/dashboard riêng của workshop.

#### 6. Xác nhận chi phí

1. Mở **Cost Explorer** và lọc theo tag/tên FoodieRecipe.
2. Kiểm tra S3, CloudFront, EC2, RDS, Rekognition, Bedrock, Secrets Manager và CloudWatch.
3. Giữ Budget Alert trong vài ngày nếu cần theo dõi chi phí đến trễ.

#### Kết quả

- Tài nguyên workshop đã được dọn theo đúng thứ tự phụ thuộc.
- Không còn S3 object, CloudFront distribution, EC2/RDS hoặc secret thử nghiệm.
- Tài nguyên dùng chung và dữ liệu cần giữ không bị ảnh hưởng.
