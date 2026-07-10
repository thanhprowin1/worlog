---
title: "Các bài Blog kỹ thuật"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Phần này liệt kê và giới thiệu các bài blog kỹ thuật mà tôi đã nghiên cứu, biên soạn và dịch thuật trong quá trình học tập. Các bài viết này phản ánh kinh nghiệm thực tế của tôi trong việc giải quyết các bài toán trên hệ sinh thái đám mây.

### [Blog 1 - Đừng để EC2 chạy xuyên đêm: Cách giảm mạnh chi phí AWS bằng Spot Instance, Auto Scaling và Scheduler](3.1-Blog1/)
Bài blog này khám phá các chiến lược tối ưu hóa chi phí đám mây cho môi trường dev/test. Bạn sẽ tìm hiểu cách tận dụng AWS Spot Instances để tiết kiệm tới 90% chi phí, cách xử lý rủi ro thu hồi máy chủ bằng việc thiết kế ứng dụng stateless, đảm bảo tính sẵn sàng cao với Auto Scaling Group (Capacity Rebalancing) và tự động hóa vòng đời tài nguyên bằng AWS Instance Scheduler.

---

### [Blog 2 - Cách chống quên tắt tài nguyên AWS: Budgets, Cost Anomaly Detection và Tagging](3.2-Blog2/)
Bài blog tập trung vào việc xây dựng nền tảng quản trị chi phí (Cost Governance) cho người mới bắt đầu dùng AWS. Nội dung hướng dẫn thiết lập 3 lớp phòng ngự tài chính: AWS Budgets để cảnh báo hạn mức dự báo, Cost Anomaly Detection dùng Machine Learning để phát hiện chi tiêu bất thường, và quy tắc Tagging nghiêm ngặt để phân loại chi phí.

---

### [Blog 3 - Thiết kế ứng dụng Stateless trên AWS: Vì sao không nên lưu file trực tiếp trong EC2?](3.3-Blog3/)
Bài blog đi sâu vào các best practices của kiến trúc cloud-native, đặc biệt là tầm quan trọng của việc xây dựng ứng dụng phi trạng thái (stateless). Bạn sẽ học cách tách biệt hoàn toàn các phân lớp dữ liệu: chuyển file tĩnh lên Amazon S3, cơ sở dữ liệu sang Amazon RDS và log hệ thống về Amazon CloudWatch, từ đó giúp hệ thống dễ dàng mở rộng (scale-out) và an toàn khi sử dụng Spot Instances.