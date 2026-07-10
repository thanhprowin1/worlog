---
title: "Worklog Tuần 2"
date: 2026-04-27
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:
* Mở rộng lớp dữ liệu: CSDL quan hệ được quản lý, NoSQL, cache và lưu trữ object nâng cao.


### Các công việc cần triển khai:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tạo instance Amazon RDS, kết nối từ EC2 trong VPC private <br> - So sánh mô hình managed DB với tự cài trên EC2 | 27/04/2026 | 27/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Thiết kế bảng DynamoDB (partition/sort key, GSI) <br> - Bổ sung lớp cache với Amazon ElastiCache | 28/04/2026 | 28/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Phân phối nội dung tĩnh qua Amazon CloudFront <br> - Gắn origin S3, cấu hình cache behavior cơ bản | 29/04/2026 | 29/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Thử quy trình di chuyển: VM Import và AWS DMS <br>  | 30/04/2026 | 30/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Lưu trữ sẵn sàng cao với EBS Multi-Attach <br>  | 01/05/2026 | 01/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được:
* **Lớp dữ liệu đa dạng:** Biết khi nào dùng RDS, DynamoDB hay cache; hiểu trade-off giữa quan hệ và NoSQL.
* **Tốc độ toàn cầu:** CloudFront đứng trước S3 giúp giảm latency cho người dùng xa region.

