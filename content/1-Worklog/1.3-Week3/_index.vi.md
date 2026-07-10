---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:
* Xây “vòng bảo vệ” quanh ứng dụng: định danh người dùng, tường lửa biên, mã hóa và phát hiện đe dọa.
* Kết nối mạng nhiều VPC và hiểu cách Lightsail đơn giản hóa điện toán khi chưa cần full VPC stack.

### Các công việc cần triển khai:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tạo Cognito User Pool cho đăng ký / đăng nhập <br> - So sánh Cognito với luồng SSO doanh nghiệp | 04/05/2026 | 04/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Gắn AWS WAF vào CloudFront; thử rule XSS/SQLi mẫu <br> - Tổng quan AWS Firewall Manager cho quản trị tập trung | 05/05/2026 | 05/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tạo và dùng customer managed key với AWS KMS <br> - Quét dữ liệu nhạy cảm bằng Amazon Macie (lab) | 06/05/2026 | 06/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Bật Amazon GuardDuty và đọc findings mẫu <br> - Nối mạng multi-VPC với Transit Gateway | 07/05/2026 | 07/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Triển khai nhanh app demo trên Amazon Lightsail <br> - Đối chiếu Lightsail với EC2 + VPC tự dựng | 08/05/2026 | 08/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được:
* **Bảo mật nhiều lớp:** Định danh (Cognito/SSO), biên (WAF), dữ liệu (KMS/Macie) và phát hiện (GuardDuty) tạo thành một chuỗi phòng thủ.
* **Mạng mở rộng:** Transit Gateway giúp hình dung topology khi có nhiều VPC / môi trường.
* **Lựa chọn điện toán:** Biết khi nào Lightsail đủ dùng và khi nào nên quay lại EC2 + VPC đầy đủ.
