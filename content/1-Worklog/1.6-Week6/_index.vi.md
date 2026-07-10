---
title: "Worklog Tuần 6"
date: 2026-05-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:
* Hoàn thiện tư duy ứng dụng hiện đại: microservices, event-driven, full-stack serverless.


### Các công việc cần triển khai:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Phân tích cách tách monolith thành microservices <br> - Thiết kế luồng Event-Driven với SQS/SNS | 25/05/2026 | 25/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Xây backend Book Store: Lambda + API Gateway + DynamoDB + S3 <br> - Deploy bằng AWS SAM; gắn Cognito cho xác thực | 26/05/2026 | 26/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Ghép frontend SPA, cấu hình auth callback Cognito <br> - Thử API GraphQL với AWS AppSync; gắn dịch vụ AI AWS vào app demo | 27/05/2026 | 27/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Thiết kế Data Lake trên S3 (raw / curated zones) <br> - Catalog bằng Glue; truy vấn serverless với Athena; dashboard QuickSight | 28/05/2026 | 28/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Ôn PostgreSQL nâng cao trên AWS (backup, performance) <br> - Vòng đời ML và huấn luyện mô hình mẫu với Amazon SageMaker | 29/05/2026 | 29/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được:
* **Ứng dụng serverless hoàn chỉnh (lab):** Từ auth Cognito → API → Lambda/DynamoDB → frontend SPA, kèm messaging và GraphQL.
* **Data & BI:** Biết tổ chức Data Lake, query bằng Athena và kể chuyện bằng QuickSight.
* **Cửa ngõ AI/ML:** Hiểu pipeline huấn luyện cơ bản trên SageMaker; PostgreSQL nâng cao bổ sung góc nhìn CSDL quan hệ production.
