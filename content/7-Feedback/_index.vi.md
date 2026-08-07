---
title: "Chia sẻ và đóng góp ý kiến"
date: 2026-06-22
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

## Đánh giá chung về quá trình thực tập tại Bootcamp First Cloud AI Journey (FCAJ)

### 1. Môi trường làm việc và học tập

Trong thời gian tham gia **Bootcamp First Cloud AI Journey (FCAJ)** từ ngày **22/06/2026** đến ngày **14/08/2026**, em cảm nhận đây là một môi trường học tập chuyên nghiệp, thân thiện và có tính thực hành cao. Mặc dù chương trình được tổ chức theo mô hình Bootcamp thay vì doanh nghiệp truyền thống, quy trình học tập, báo cáo tiến độ, làm việc nhóm và phát triển dự án được xây dựng tương đối gần với môi trường làm việc thực tế.

Mentor, Teaching Assistant (TA), Team Admin và các học viên luôn sẵn sàng hỗ trợ nhau khi gặp khó khăn. Trong quá trình xây dựng **FoodieRecipe**, em có thể trao đổi những vấn đề liên quan đến kiến trúc, tích hợp dịch vụ AWS, luồng xử lý ảnh và cách trình bày tài liệu. Không khí cởi mở giúp em mạnh dạn đặt câu hỏi, chia sẻ cách làm và tiếp nhận các góc nhìn khác nhau.

Điều em ấn tượng nhất là chương trình khuyến khích học viên chủ động nghiên cứu trước khi yêu cầu hỗ trợ. Khi gặp lỗi trong luồng upload ảnh, phân quyền S3 hoặc gọi dịch vụ AI, em được định hướng cách đọc log, kiểm tra cấu hình và tham khảo tài liệu chính thức thay vì chỉ nhận một đáp án có sẵn. Nhờ đó, em dần hình thành thói quen tự tìm hiểu và giải quyết vấn đề có hệ thống.

### 2. Sự hỗ trợ của Mentor, Teaching Assistant và Team Admin

Trong suốt quá trình thực tập, em nhận được sự hướng dẫn tận tình từ mentor và TA. Khi nhóm gặp khó khăn trong việc xác định workflow hoặc lựa chọn dịch vụ phù hợp cho FoodieRecipe, mentor hỗ trợ phân tích yêu cầu, đặt câu hỏi gợi mở và góp ý để nhóm tự hoàn thiện phương án.

Các góp ý về Amazon S3, Amazon Rekognition, Amazon Bedrock và Amazon CloudFront giúp em hiểu rõ hơn rằng một tính năng AI hoàn chỉnh không chỉ dừng ở việc gọi model. Hệ thống còn phải quan tâm đến quyền truy cập, định dạng dữ liệu, trạng thái xử lý, chất lượng kết quả, khả năng xử lý lỗi, trải nghiệm xác nhận của người dùng và chi phí vận hành.

Mentor cũng chia sẻ nhiều kinh nghiệm thực tế về cách trình bày kiến trúc, quản lý secret, áp dụng IAM theo nguyên tắc least privilege và theo dõi hệ thống bằng CloudWatch. Những chia sẻ này giúp em hiểu rõ hơn trách nhiệm của một kỹ sư khi xây dựng hệ thống Cloud, thay vì chỉ tập trung vào việc hoàn thành chức năng.

Bên cạnh đó, Team Admin hỗ trợ nhanh chóng trong việc cung cấp tài liệu, cập nhật lịch học, tổ chức sự kiện và giải đáp các vấn đề liên quan đến chương trình. Nhờ sự phối hợp của mentor, TA và Team Admin, em có thể tập trung tốt hơn vào việc học tập, thực hiện dự án và hoàn thiện báo cáo.

### 3. Sự phù hợp giữa nội dung thực tập và chuyên ngành học

Theo em, nội dung thực tập tại FCAJ phù hợp với chuyên ngành Công nghệ thông tin. Những kiến thức nền tảng về lập trình web, mạng máy tính, cơ sở dữ liệu và phân tích hệ thống được áp dụng trực tiếp khi xây dựng FoodieRecipe.

Trong dự án, em sử dụng **Next.js** trong thư mục `web` để phát triển Frontend với chức năng quản lý công thức, lượt thích và bình luận; **NestJS** trong thư mục `api` để xây dựng Backend. Hệ thống sử dụng **Amazon RDS for PostgreSQL** để lưu dữ liệu quan hệ; **Amazon S3** để lưu ảnh; **Amazon Rekognition** để nhận diện và kiểm duyệt ảnh; **Amazon Bedrock** để gợi ý nội dung công thức; và **Amazon CloudFront** để phân phối ảnh với tốc độ tốt hơn.

Em triển khai kiến trúc sản phẩm hoàn chỉnh với Amazon EC2, Docker, Nginx, Amazon RDS, AWS Secrets Manager, IAM và Amazon CloudWatch cho việc vận hành, bảo mật và giám sát. Công việc bao gồm xây dựng `web`, `api`, triển khai Backend lên EC2, kết nối RDS, phát hành Frontend qua S3/CloudFront và hoàn thiện luồng ảnh S3–Rekognition–Bedrock–CloudFront.

Việc kết hợp kiến thức lập trình với Cloud và Generative AI giúp em nhìn rõ hơn mối liên hệ giữa các môn học ở trường và yêu cầu thực tế của một sản phẩm phần mềm. Đây là những kiến thức có tính ứng dụng cao và bổ sung cho các nội dung mà em chưa có nhiều cơ hội thực hành chuyên sâu trên giảng đường.

### 4. Cơ hội học hỏi và phát triển kỹ năng

Khoảng thời gian tham gia Bootcamp mang đến cho em nhiều cơ hội phát triển cả kiến thức chuyên môn lẫn kỹ năng làm việc. Thông qua FoodieRecipe, em hiểu rõ hơn quy trình từ phân tích yêu cầu, thiết kế kiến trúc, phát triển tính năng, kiểm thử đến viết tài liệu và báo cáo kết quả.

Về chuyên môn, em học được cách tạo S3 pre-signed URL để upload trực tiếp, tổ chức object theo prefix `uploads/` và `delivery/`, kiểm soát quyền truy cập bằng IAM, sử dụng Rekognition để kiểm duyệt và nhận diện ảnh, gửi dữ liệu đến Bedrock và phân phối nội dung qua CloudFront. Em cũng hiểu rõ hơn cách quản lý bốn trạng thái ảnh gồm `pending`, `processing`, `completed` và `failed` để theo dõi luồng xử lý.

Về kỹ năng, em được rèn luyện cách lập kế hoạch, phân chia nhiệm vụ, theo dõi tiến độ và phối hợp với các thành viên trong nhóm. Khi gặp lỗi, em học cách đọc thông báo, kiểm tra từng bước của workflow, tìm kiếm tài liệu kỹ thuật và xác định nguyên nhân trước khi đề xuất phương án xử lý.

Ngoài ra, kỹ năng đọc tài liệu tiếng Anh, viết Worklog, xây dựng Proposal, biên soạn Workshop song ngữ, thiết kế sơ đồ kiến trúc và trình bày kết quả của em cũng được cải thiện đáng kể. Những kỹ năng này là nền tảng quan trọng để em thích nghi tốt hơn với môi trường làm việc chuyên nghiệp sau khi tốt nghiệp.

### 5. Văn hóa chia sẻ và tinh thần đồng đội

Điều em đánh giá cao tại FCAJ là văn hóa chia sẻ kiến thức giữa các thành viên. Mọi người sẵn sàng trao đổi về lỗi gặp phải, tài liệu hữu ích và kinh nghiệm thực hành. Điều này giúp mỗi cá nhân không chỉ hoàn thành phần việc của mình mà còn hiểu thêm về các thành phần khác trong toàn bộ hệ thống.

Trong quá trình làm việc nhóm, các thành viên có trách nhiệm với phần công việc được phân công và cùng thống nhất kiến trúc, workflow cũng như cách tích hợp giữa Frontend, Backend và AWS. Khi một thành viên gặp khó khăn, mọi người cùng trao đổi để tìm phương án phù hợp thay vì tách rời từng phần việc.

Mentor luôn tạo điều kiện để các thành viên trình bày ý tưởng, phản biện và đóng góp ý kiến. Điều này giúp em cảm thấy được tôn trọng, nâng cao tinh thần trách nhiệm và có thêm động lực để hoàn thiện phần việc của mình. Dù thời gian thực tập không dài, em vẫn cảm nhận rõ sự đoàn kết, nghiêm túc và tinh thần học hỏi của cộng đồng FCAJ.

## Cảm nhận cá nhân sau quá trình thực tập

Sau thời gian tham gia Bootcamp First Cloud AI Journey, em cảm thấy đây là một trải nghiệm ý nghĩa đối với quá trình học tập và định hướng nghề nghiệp của bản thân. Điều em hài lòng nhất là được trực tiếp áp dụng kiến thức vào FoodieRecipe — một sản phẩm có sự kết hợp giữa phát triển web, Cloud và AI — thay vì chỉ học từng dịch vụ một cách riêng lẻ.

Quá trình thực hiện dự án giúp em hiểu rằng việc xây dựng một tính năng thực tế đòi hỏi nhiều yếu tố phối hợp: dữ liệu phải được lưu trữ an toàn, quyền truy cập cần được giới hạn, kết quả AI phải được kiểm tra, người dùng cần có quyền chỉnh sửa nội dung đề xuất và hệ thống phải có khả năng theo dõi khi xảy ra lỗi. Đây là những bài học quan trọng giúp em có cách nhìn toàn diện hơn khi phát triển phần mềm.

Bên cạnh kiến thức chuyên môn, em còn cải thiện kỹ năng làm việc nhóm, quản lý tiến độ, đọc tài liệu, viết báo cáo, trình bày ý tưởng và tiếp nhận phản hồi. Những kinh nghiệm này giúp em tự tin hơn khi chuẩn bị bước vào môi trường làm việc sau khi tốt nghiệp.

## Đề xuất và đóng góp ý kiến

Theo quan điểm cá nhân, chương trình đã được xây dựng tương đối bài bản với lộ trình rõ ràng và sự hỗ trợ tích cực từ mentor, TA và Team Admin. Để chương trình tiếp tục hoàn thiện, em xin đề xuất một số ý kiến:

- Tổ chức thêm các buổi chia sẻ từ Cloud Engineer, DevOps Engineer, Solution Architect hoặc AI Engineer đang làm việc tại doanh nghiệp.
- Bổ sung một buổi review kiến trúc giữa kỳ để phát hiện sớm sự chưa thống nhất giữa Proposal, source code và Workshop.
- Cung cấp thêm ví dụ thực tế về IAM least privilege, CloudWatch Logs, xử lý lỗi AWS SDK và tối ưu chi phí khi sử dụng dịch vụ AI.
- Bổ sung hướng dẫn về unit test, integration test và cách mock các dịch vụ AWS để học viên kiểm thử mà không phát sinh nhiều chi phí.
- Cung cấp checklist cho từng giai đoạn gồm mục tiêu, tài nguyên cần tạo, chi phí dự kiến, tiêu chí hoàn thành và bước cleanup.
- Tăng cơ hội để các nhóm trình bày sản phẩm và trao đổi về những phương án kiến trúc khác nhau.

Nếu có bạn bè hoặc sinh viên quan tâm đến Cloud Computing, DevOps hoặc AI, em sẵn sàng giới thiệu Bootcamp FCAJ vì chương trình có nội dung thực tiễn, cộng đồng hỗ trợ tích cực và tạo điều kiện để học viên phát triển cả kiến thức chuyên môn lẫn kỹ năng làm việc.

Trong tương lai, nếu có cơ hội, em mong muốn tiếp tục tham gia các chương trình chuyên sâu của FCAJ về AWS Solution Architecture, DevOps, bảo mật Cloud hoặc Generative AI. Em cũng muốn tiếp tục hoàn thiện FoodieRecipe theo hướng xử lý bất đồng bộ, tăng cường kiểm thử, theo dõi chi phí và cải thiện khả năng quan sát hệ thống.

Cuối cùng, em xin gửi lời cảm ơn chân thành đến các anh chị mentor, Teaching Assistant, Team Admin và Ban tổ chức Bootcamp First Cloud AI Journey đã tận tình hướng dẫn, hỗ trợ và tạo điều kiện để em học tập trong một môi trường chuyên nghiệp. Khoảng thời gian thực tập tại FCAJ đã giúp em trưởng thành hơn về kiến thức chuyên môn, kỹ năng làm việc và tư duy giải quyết vấn đề. Đây sẽ là nền tảng quan trọng để em tiếp tục phát triển trong lĩnh vực Cloud Computing và trí tuệ nhân tạo trong tương lai.
