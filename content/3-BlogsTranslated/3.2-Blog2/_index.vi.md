---
title: "Blog 2: Cách chống quên tắt tài nguyên AWS"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Cách chống quên tắt tài nguyên AWS: Budgets, Cost Anomaly Detection và Tagging

Khi mới làm quen với AWS, nỗi lo lớn nhất của nhiều bạn không phải là code lỗi, mà là sợ bị trừ tiền vì quên tắt tài nguyên. Kịch bản rất quen thuộc: bạn tạo EC2, bật RDS để test, rồi vô tình quên mất suốt cả tuần. Đến cuối tháng mở mục Billing ra mới phát hiện chi phí đã tăng mạnh. 

Để hạn chế tình huống này, chúng ta cần thiết lập hàng rào cảnh báo sớm bằng 3 công cụ: **AWS Budgets, AWS Cost Anomaly Detection và Cost Allocation Tags.**

---

## Những Điểm Nổi Bật Của Giải Pháp

### 1. Tạo hàng rào cảnh báo sớm với AWS Budgets
* **Vấn đề:** Kiểm tra hóa đơn thủ công bằng mắt mỗi ngày là việc bất khả thi.
* **Giải pháp:** Sử dụng **AWS Budgets** để thiết lập ngân sách cố định, ví dụ đặt hạn mức $10/tháng.
* **Điểm tối ưu:** Dịch vụ này cảnh báo dựa trên cả chi phí thực tế (Actual) và chi phí dự báo (Forecasted). Dù mới tiêu hết $2 nhưng nếu hệ thống dự báo cuối tháng vượt ngưỡng $10, bạn vẫn nhận được cảnh báo sớm.

![Bảng điều khiển AWS Budgets](/worlog/images/Blog2.jpg)

### 2. Phát hiện bất thường bằng AWS Cost Anomaly Detection
* **Rủi ro cốt lõi:** Nếu bạn bị lộ Access Key hoặc chạy nhầm script tạo hàng loạt dịch vụ đắt tiền, chi phí sẽ tăng vọt (Spike) cực nhanh trước khi kịp kích hoạt Budget Alert.
* **Giải pháp:** Kích hoạt **AWS Cost Anomaly Detection**. Dịch vụ này ứng dụng Machine Learning để liên tục theo dõi mô hình chi tiêu hàng ngày và gửi cảnh báo khi phát hiện các khoản tiền bất thường.

### 3. Phân loại và bóc tách chi phí bằng Tagging
* **Giải pháp:** Xây dựng quy tắc đặt nhãn (**Tagging**) nghiêm ngặt cho mọi tài nguyên ngay từ khi khởi tạo (ví dụ: `Project = DoAnTotNghiep`, `Environment = Dev`).
* **Lưu ý kỹ thuật:** Các tag này bắt buộc phải được kích hoạt trong mục *Cost Allocation Tags* của console Billing thì mới dùng được cho việc phân bổ chi phí.

---

## Kết Luận
Quản trị chi phí trên Cloud (FinOps) không phải là việc đợi đến cuối tháng mới kiểm tra và hối hận. Với người mới học AWS, nên thiết lập tối thiểu 3 lớp phòng ngự trên để một tài khoản AWS luôn an toàn về mặt kinh tế.