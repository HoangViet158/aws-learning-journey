---
title: "Tự đánh giá"
date: 2026-06-22
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Trong suốt thời gian thực tập tại **BOOTCAMP FIRST CLOUD AI JOURNEY (FCAJ)**, từ ngày **22/06/2026** đến ngày **14/08/2026**, em đã có cơ hội học hỏi, rèn luyện và áp dụng những kiến thức đã được học trên giảng đường vào môi trường làm việc và học tập thực tế. Đây là khoảng thời gian giúp em tiếp cận gần hơn với quy trình phân tích, thiết kế và phát triển một sản phẩm trên nền tảng điện toán đám mây, đồng thời hiểu rõ hơn về vai trò của Cloud và AI trong một hệ thống web hiện đại.

Trong quá trình tham gia Bootcamp, em thực hiện dự án **FoodieRecipe** — ứng dụng chia sẻ công thức nấu ăn có tích hợp AI. Sản phẩm cho phép người dùng quản lý và tìm kiếm công thức, thích hoặc bỏ thích, tạo bình luận, tải ảnh món ăn lên hệ thống, nhận diện nội dung ảnh và nhận các gợi ý hỗ trợ tạo công thức. Kiến trúc tổng thể sử dụng **Next.js** trong thư mục `web` cho Frontend, **NestJS** trong thư mục `api` cho Backend, **Amazon RDS for PostgreSQL** để lưu dữ liệu quan hệ, cùng các dịch vụ AWS phục vụ lưu trữ, xử lý AI, phân phối nội dung, bảo mật và giám sát.

Em thực hiện toàn bộ chức năng của sản phẩm, từ xác thực, quản lý công thức, lượt thích, bình luận đến luồng xử lý ảnh thông minh. Em xây dựng cơ chế để Backend cấp **S3 pre-signed URL**, cho phép trình duyệt tải ảnh trực tiếp lên **Amazon S3** mà không truyền toàn bộ tệp qua API. Sau khi upload hoàn tất, **Amazon Rekognition** được sử dụng để nhận diện nhãn và kiểm duyệt nội dung ảnh. Những ảnh hợp lệ cùng kết quả nhận diện được chuyển đến mô hình thông qua **Amazon Bedrock** để gợi ý tên món ăn, mô tả, nguyên liệu và tag. Người dùng có thể xem lại, chỉnh sửa và xác nhận nội dung do AI đề xuất trước khi lưu công thức.

Đối với việc phân phối ảnh, em tìm hiểu cách sử dụng **Amazon CloudFront** với S3 private origin và Origin Access Control nhằm cải thiện tốc độ truy cập, tận dụng cơ chế cache và hạn chế việc truy cập trực tiếp vào S3. Luồng ảnh được tổ chức trong một S3 image bucket với hai prefix: `uploads/` dành cho ảnh gốc và `delivery/` dành cho ảnh đã xử lý hoàn tất. Trạng thái xử lý ảnh được rút gọn thành `pending`, `processing`, `completed` và `failed` để thuận tiện theo dõi và xử lý lỗi.

Bên cạnh đó, em thực hiện phân tích yêu cầu, xây dựng workflow, thiết kế kiến trúc, phát triển các chức năng Next.js/NestJS, viết tài liệu Proposal và Workshop song ngữ, kiểm thử và hoàn thiện báo cáo theo góp ý. Em tạo **Amazon RDS**, quản lý secret bằng **AWS Secrets Manager**, đóng gói Backend bằng **Docker**, triển khai lên **Amazon EC2** sau **Nginx**, gắn **IAM Role** và cấu hình **Amazon CloudWatch** để thu thập log, metrics và cảnh báo. Nhờ đó, em có cơ hội thực hiện toàn bộ quy trình từ phát triển đến triển khai và vận hành FoodieRecipe.

Thông qua quá trình thực hiện FoodieRecipe, em đã cải thiện đáng kể kiến thức về Cloud Computing, khả năng tích hợp dịch vụ AWS bằng SDK, kỹ năng đọc hiểu tài liệu kỹ thuật, thiết kế luồng dữ liệu, quản lý quyền truy cập và xử lý lỗi. Em cũng hiểu rõ hơn về nguyên tắc least privilege trong IAM, bảo vệ S3 bucket, quản lý cấu hình bằng biến môi trường, sử dụng URL có thời hạn và theo dõi hệ thống bằng CloudWatch. Đồng thời, em rèn luyện thêm kỹ năng làm việc nhóm, quản lý tiến độ, trình bày ý tưởng, viết báo cáo và giải quyết các vấn đề phát sinh trong quá trình tích hợp nhiều dịch vụ.

Trong suốt thời gian thực tập, em luôn cố gắng hoàn thành các nhiệm vụ được giao đúng tiến độ, chủ động nghiên cứu tài liệu chính thức của AWS khi gặp khó khăn và tích cực trao đổi với mentor cũng như các thành viên trong nhóm để thống nhất giải pháp. Mặc dù vẫn còn nhiều kiến thức cần tiếp tục học hỏi, em luôn giữ tinh thần cầu tiến, sẵn sàng tiếp nhận góp ý và điều chỉnh sản phẩm cũng như tài liệu để nâng cao chất lượng công việc.

Để nhìn nhận một cách khách quan về quá trình thực tập, em xin tự đánh giá bản thân theo các tiêu chí sau:

| STT | Tiêu chí | Mô tả | Tốt | Khá | Trung bình |
| --- | --- | --- | :---: | :---: | :---: |
| 1 | **Kiến thức và kỹ năng chuyên môn** | Áp dụng AWS Cloud, Next.js, NestJS, PostgreSQL và AI vào toàn bộ FoodieRecipe | ☐ | ✅ | ☐ |
| 2 | **Khả năng học hỏi** | Chủ động tìm hiểu S3, Rekognition, Bedrock, CloudFront và tài liệu AWS | ✅ | ☐ | ☐ |
| 3 | **Tinh thần chủ động** | Chủ động nhận nhiệm vụ, phân tích workflow và tìm giải pháp trước khi trao đổi với mentor | ✅ | ☐ | ☐ |
| 4 | **Tinh thần trách nhiệm** | Hoàn thành phần việc và tài liệu được giao đúng tiến độ, bảo đảm nội dung nhất quán | ✅ | ☐ | ☐ |
| 5 | **Kỷ luật** | Tham gia đầy đủ các buổi học, họp nhóm và tuân thủ quy định của Bootcamp | ☐ | ✅ | ☐ |
| 6 | **Tinh thần cầu tiến** | Tiếp thu góp ý, điều chỉnh kiến trúc, workflow và tài liệu để hoàn thiện sản phẩm | ✅ | ☐ | ☐ |
| 7 | **Kỹ năng giao tiếp** | Trao đổi yêu cầu, tiến độ và vấn đề kỹ thuật với mentor và thành viên trong nhóm | ☐ | ✅ | ☐ |
| 8 | **Khả năng làm việc nhóm** | Phối hợp các phần Frontend, Backend và AWS để bảo đảm luồng xử lý thống nhất | ✅ | ☐ | ☐ |
| 9 | **Thái độ và tác phong** | Tôn trọng mentor, hỗ trợ thành viên và nghiêm túc trong quá trình học tập, thực hiện dự án | ✅ | ☐ | ☐ |
| 10 | **Tư duy giải quyết vấn đề** | Phân tích lỗi upload, quyền truy cập, kết quả AI và đề xuất phương án xử lý phù hợp | ☐ | ✅ | ☐ |
| 11 | **Đóng góp cho dự án** | Hoàn thành ứng dụng, hạ tầng deploy, luồng ảnh AI, Proposal, Workshop và tài liệu dự án | ☐ | ✅ | ☐ |
| 12 | **Đánh giá tổng thể** | Hoàn thành mục tiêu và phạm vi công việc trong thời gian thực tập tại FCAJ | ☐ | ✅ | ☐ |

## Những điểm cần cải thiện

Qua quá trình thực tập, em nhận thấy bản thân vẫn còn một số điểm cần tiếp tục hoàn thiện để đáp ứng tốt hơn yêu cầu của công việc trong tương lai:

- Tiếp tục nâng cao kiến thức chuyên sâu về kiến trúc AWS, bảo mật, IAM least privilege, khả năng mở rộng và tối ưu chi phí cho các hệ thống có tích hợp AI.
- Rèn luyện thêm kỹ năng viết unit test và integration test cho NestJS, đặc biệt với các thành phần sử dụng AWS SDK và dịch vụ bên ngoài.
- Tìm hiểu cách xử lý bất đồng bộ bằng hàng đợi để luồng Rekognition và Bedrock ổn định hơn khi số lượng ảnh tăng cao.
- Cải thiện khả năng phân tích log, theo dõi metrics, thiết lập cảnh báo và xử lý sự cố bằng Amazon CloudWatch.
- Nâng cao kỹ năng giao tiếp và trình bày ý tưởng trong các buổi họp nhóm cũng như khi báo cáo với mentor để truyền đạt nội dung rõ ràng, ngắn gọn và thuyết phục hơn.
- Cải thiện kỹ năng quản lý thời gian, chia nhỏ đầu việc và đánh giá rủi ro sớm khi thực hiện dự án có nhiều thành phần tích hợp.
- Tiếp tục học hỏi về Cloud Computing, DevOps, CI/CD và Generative AI để hiểu đầy đủ vòng đời phát triển và vận hành một sản phẩm thực tế.

Nhìn chung, em đánh giá quá trình thực tập tại **BOOTCAMP FIRST CLOUD AI JOURNEY** là một trải nghiệm rất bổ ích. Chương trình không chỉ giúp em củng cố kiến thức chuyên môn về AWS mà còn tạo điều kiện để em áp dụng Cloud và AI vào dự án FoodieRecipe, rèn luyện kỹ năng làm việc thực tế và xác định rõ hơn định hướng phát triển bản thân trong lĩnh vực Cloud Computing và trí tuệ nhân tạo trong tương lai.
