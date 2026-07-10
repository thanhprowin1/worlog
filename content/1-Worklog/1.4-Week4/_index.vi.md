---
title: "Worklog Tuần 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Chuyển từ bấm Console sang tự động hóa: Lambda, Systems Manager, IaC (CloudFormation & CDK).
* Kết hợp giám sát nâng cao và FinOps để vận hành bền vững về chi phí.

### Các công việc cần triển khai:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết Lambda xử lý sự kiện S3 / schedule EventBridge <br> - Truy cập EC2 an toàn qua Systems Manager Session Manager (không mở SSH public) | 11/05/2026 | 11/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Khai báo stack CloudFormation (VPC + EC2 tối giản) <br> - So sánh template YAML với thao tác thủ công tuần 1 | 12/05/2026 | 12/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Dựng cùng stack bằng AWS CDK (TypeScript/Python) <br> - Cài AWS Toolkit trên VS Code để deploy từ IDE | 13/05/2026 | 13/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Dashboard CloudWatch nâng cao; thử Grafana datasource <br> - Bật VPC Flow Logs; đặt lịch snapshot bằng EBS Data Lifecycle Manager | 14/05/2026 | 14/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Phân tích Savings Plans và Reserved Instances cho lab <br> - Truy vấn dữ liệu chi phí bằng Glue Crawler + Athena | 15/05/2026 | 15/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được:
* **Tự động hóa vận hành:** Lambda thay việc lặp lại; SSM giúp quản trị máy chủ không cần bastion mở internet.
* **Hạ tầng tái tạo được:** Cùng một stack có thể dựng lại bằng CloudFormation hoặc CDK, gắn với source control.
* **Quan sát + chi phí:** Flow Logs / CloudWatch cho visibility; Savings Plans/RI và Athena giúp đọc bill có hệ thống.
