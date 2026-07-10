---
title: "Blog 3: Thiết kế ứng dụng Stateless trên AWS"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Thiết kế ứng dụng Stateless trên AWS: Vì sao không nên lưu file trực tiếp trong EC2?

Khi triển khai một ứng dụng Web trên một server EC2, thói quen phổ biến nhất là lưu trữ mọi thứ chung một chỗ: Source code, ảnh user upload trên EBS volume, và database cài trực tiếp lên chính server đó.

Tuy nhiên, thách thức sẽ xuất hiện khi hệ thống cần mở rộng (Scale-out) hoặc áp dụng EC2 Spot Instance (có thể bị thu hồi bất kỳ lúc nào). Bản chất của EC2 là tài nguyên mang tính tạm thời. Để giải quyết triệt để rủi ro mất mát dữ liệu và sẵn sàng tự động co giãn, ứng dụng cần được thiết kế theo hướng **stateless (phi trạng thái)**: tách file, database và log sang các dịch vụ chuyên biệt.

---

## Những Điểm Nổi Bật Của Giải Pháp

### 1. Tách biệt tầng lưu trữ file tĩnh với Amazon S3
* **Vấn đề:** Khi mở rộng ra nhiều EC2, ảnh upload lên Server A sẽ không tồn tại trên Server B và Server C.
* **Giải pháp:** Không lưu file upload quan trọng lâu dài trên ổ đĩa EC2. Hãy upload kết quả cuối cùng lên **Amazon S3**.
* **Kết quả:** Ổ đĩa của EC2 giờ chỉ chứa source code tĩnh. Toàn bộ các máy chủ truy cập chung vào kho lưu trữ S3 tập trung để lấy và hiển thị file.

![Sơ đồ kiến trúc ứng dụng Stateless](/images/Blog3.jpg)

### 2. Độc lập tầng dữ liệu có cấu trúc với Amazon RDS
* **Vấn đề:** Cài đặt Database chung với Web server sẽ ngốn rất nhiều tài nguyên và rủi ro cao nếu hệ điều hành bị crash.
* **Giải pháp:** Tách biệt hoàn toàn tầng dữ liệu bằng cách sử dụng **Amazon RDS**.
* **Kết quả:** EC2 chỉ giữ vai trò xử lý logic (Compute) và truy vấn dữ liệu sang RDS. RDS hỗ trợ automated backup và Multi-AZ tăng khả năng sẵn sàng.

### 3. Tập trung hóa dữ liệu Log hệ thống bằng Amazon CloudWatch Logs
* **Vấn đề:** Lập trình viên thường SSH vào EC2 để đọc log. Nếu server bị Auto Scaling thu hồi, dấu vết log sẽ mất hoàn toàn.
* **Giải pháp:** Cấu hình **CloudWatch Agent** để đẩy log ứng dụng và hệ thống về CloudWatch Logs gần thời gian thực.
* **Kết quả:** Kể cả khi EC2 bị khai tử, toàn bộ lịch sử hoạt động vẫn được lưu trữ an toàn, độc lập trên Cloud để phân tích.

---

## Kết Luận
Sự bền bỉ của kiến trúc không nằm ở việc cố giữ cho một con EC2 sống mãi, mà nằm ở khả năng **Tách biệt hoàn toàn các phân lớp dữ liệu (Decoupling Architecture)**. Khi ứng dụng đã đạt trạng thái Stateless, bạn có thể tự tin bật Auto Scaling Group hoặc dùng Spot Instance để tiết kiệm chi phí mà không còn sợ hãi việc mất dữ liệu.