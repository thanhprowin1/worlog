---
title: "Worklog Tuần 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Đóng gói ứng dụng bằng container và điều phối trên ECS/Fargate cũng như Kubernetes (EKS).
* Tự động hóa đường đi từ code → image → cluster bằng CI/CD; làm quen workflow và lưu trữ lai.

### Các công việc cần triển khai:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Build image local và đẩy lên Amazon ECR <br> - Chạy service trên Amazon ECS với launch type Fargate | 18/05/2026 | 18/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Dựng pipeline CodePipeline: source → build → deploy ECS <br> - Mô tả cluster/service ECS bằng CDK | 19/05/2026 | 19/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tạo cluster Amazon EKS và deploy một workload mẫu (kubectl) <br> - Dùng EKS Blueprints để chuẩn hóa bootstrap cluster | 20/05/2026 | 20/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Mở rộng CI/CD cho ứng dụng trên EKS <br> - Tổng quan Red Hat OpenShift Service on AWS (ROSA) | 21/05/2026 | 21/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Điều phối multi-step workflow với Step Functions <br> - Kết nối lưu trữ lai qua Storage Gateway / Amazon FSx | 22/05/2026 | 22/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được:
* **Container thực chiến:** Image trên ECR, service chạy Fargate; không còn phụ thuộc máy ảo “cài tay”.
* **Hai mô hình điều phối:** Phân biệt ECS (đơn giản hơn) và EKS/K8s (linh hoạt, phức tạp hơn); biết ROSA nằm ở đâu trong bức tranh enterprise.
* **Pipeline & workflow:** CodePipeline nối code với cluster; Step Functions + hybrid storage mở rộng khả năng tích hợp on-prem / multi-step.
