---
title: "Blog 1: Đừng để EC2 chạy xuyên đêm: Giảm mạnh chi phí AWS"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Đừng để EC2 chạy xuyên đêm: Cách mình giảm mạnh chi phí AWS bằng Spot Instance + Auto Scaling + Scheduler

Khi triển khai các dự án học tập, đồ án hoặc môi trường thử nghiệm trên AWS, nhiều bạn thường có thói quen vào EC2 Console, bấm tạo một Instance và để cấu hình mặc định là On-Demand. Cách này đơn giản, dễ dùng, nhưng nếu để server chạy xuyên đêm hoặc xuyên cuối tuần thì chi phí có thể tăng lên khá nhanh.

Việc duy trì các server On-Demand cho các môi trường thử nghiệm (Development/Staging) không yêu cầu uptime 24/7 là một sự lãng phí lớn. Để giải quyết bài toán này, dự án của mình ứng dụng giải pháp **Tối ưu hóa dung lượng dư thừa (Spot Instances)** kết hợp với kiến trúc chịu lỗi linh hoạt để cắt giảm chi phí đến mức tối đa.

---

## Những Điểm Nổi Bật Của Giải Pháp

### 1. Chiến lược săn tài nguyên giá rẻ với AWS Spot Instances
* **Vấn đề:** Giá thuê EC2 On-Demand cố định theo giờ khá cao.
* **Giải pháp:** Tận dụng lượng tài nguyên phần cứng dư thừa chưa có người thuê của AWS thông qua **Spot Instances**. AWS bán lại lượng dung lượng này với mức giảm tới 90% so với giá On-Demand.
* **Kết quả:** Chi phí EC2 có thể giảm rất mạnh, từ vài chục USD/tháng xuống chỉ còn vài USD, giúp bạn thoải mái test lab mà không lo cạn tiền.

### 2. Cơ chế phòng ngự chủ động: Đón đầu "Cú thu hồi" từ AWS
* **Rủi ro cốt lõi:** AWS có thể thu hồi Spot capacity bất kỳ lúc nào và người dùng chỉ có tối đa 2 phút để xử lý.
* **Giải pháp (Thiết kế Stateless):** Điểm mấu chốt là tuyệt đối không lưu dữ liệu quan trọng trực tiếp trên ổ đĩa cục bộ của EC2. Toàn bộ file đẩy sang Amazon S3, Database chạy trên Amazon RDS, và Log đẩy ra CloudWatch Logs.
* **Kết quả:** Ứng dụng hoàn toàn mang tính chất **Stateless**, tắt đi và khởi động lại nhanh chóng mà không mất dữ liệu.

![Sơ đồ kiến trúc Stateless](/worlog/images/Blog1.jpg)

### 3. Tự động hóa khôi phục hạ tầng bằng EC2 Auto Scaling
Mình đặt EC2 Spot vào trong một **Auto Scaling Group (ASG)** và kích hoạt tính năng **Capacity Rebalancing**. 
ASG sẽ chủ động phản ứng ngay khi nhận được tín hiệu `rebalance recommendation` từ AWS, tự động tìm kiếm một Availability Zone khác hoặc loại instance tương đương để khởi tạo máy chủ mới thay thế.

### 4. Quản lý bật/tắt thông minh với AWS Instance Scheduler
Hệ thống tích hợp thêm **Instance Scheduler on AWS** để tự động hóa tắt (Stop) toàn bộ server vào lúc 19h tối và tự động bật lại vào lúc 7h sáng hôm sau (thứ 2 - thứ 6), ngủ đông hoàn toàn vào cuối tuần. Điều này giúp loại bỏ triệt để tình trạng "quên tắt server".

---

## Kết Luận
Tối ưu chi phí trên Cloud (FinOps) không chỉ đơn giản là chọn instance cấu hình thấp. Một kiến trúc tốt cần biết chọn đúng pricing model, thiết kế ứng dụng chịu lỗi (Design for Failure), lưu dữ liệu đúng nơi và tự động hóa vòng đời tài nguyên.