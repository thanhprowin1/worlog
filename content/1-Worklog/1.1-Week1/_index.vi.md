---
title: "Worklog Tuần 1"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:
* Dựng môi trường làm việc AWS và hiểu cách tài khoản, quyền hạn, chi phí gắn với nhau.


### Các công việc cần triển khai:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Cài đặt và cấu hình AWS CLI, làm quen Console <br> - Tạo ngân sách và cảnh báo chi phí với AWS Budgets | 20/04/2026 | 20/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tạo user, group, policy và role với IAM <br> - Áp dụng nguyên tắc least privilege cho tài khoản lab | 21/04/2026 | 21/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Thiết kế VPC: subnet public/private, route table, IGW/NAT <br> - Kiểm soát traffic bằng Security Group và NACL | 22/04/2026 | 22/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Launch EC2, gắn key pair và bootstrap script cơ bản <br> - Cấu hình Auto Scaling Group theo metric CPU | 23/04/2026 | 23/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Tạo bucket S3, bật versioning và static website hosting <br> - Trỏ domain demo bằng Route 53; theo dõi metric bằng CloudWatch | 24/04/2026 | 24/04/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được:
* **Môi trường lab sẵn sàng:** CLI hoạt động, IAM được siết quyền, Budgets đã bật cảnh báo để không “cháy” credit.
* **Hạ tầng tối thiểu chạy được:** VPC + EC2 (có scale) + S3 + DNS, kèm dashboard CloudWatch theo dõi sức khỏe tài nguyên.
* **Tư duy vận hành ban đầu:** Biết mỗi resource gắn với quyền, mạng và chi phí — nền cho các tuần sau.
