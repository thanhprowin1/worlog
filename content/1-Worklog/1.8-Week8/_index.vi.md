---
title: "Worklog Tuần 8"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:
* Dựng khung hạ tầng backend: API Gateway + Lambda AI Processor và hoàn thiện schema DynamoDB.
* Đồng bộ API contract với frontend và AI team để tuần sau ráp nối trơn tru.

### Các công việc đã hoàn thành:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Trạng thái |
| --- | --- | --- | --- | --- |
| 2 | - Đồng bộ API contract: định nghĩa request/response cho `/recommendation` <br> - Thống nhất format payload giữa frontend, AI Processor | 08/06/2026 | 08/06/2026 | Hoàn thành |
| 3 | - Tạo API Gateway REST API `wakan-api`; gắn Cognito Authorizer <br> - Tạo resource `/recommendation` với method POST, trỏ đến Lambda Orchestrator | 09/06/2026 | 09/06/2026 | Hoàn thành |
| 4 | - Hoàn thiện chi tiết schema DynamoDB: Users/Subscriptions (PK userId, attributes tier/usedToday/usedThisMonth/resetAt) <br> - RecommendationCache (PK cacheKey, TTL expiresAt) | 10/06/2026 | 10/06/2026 | Hoàn thành |
| 5 | - Viết skeleton Lambda AI Processor: parse event từ API Gateway, extract userId + tier từ Cognito claims <br> - Deploy lên AWS, test bằng Console Test event | 11/06/2026 | 11/06/2026 | Hoàn thành |
| 6 | - Gắn IAM policy cho Lambda: dynamodb:GetItem/PutItem/UpdateItem + lambda:InvokeFunction <br> - Ghi log bước parse và extract claim vào CloudWatch | 12/06/2026 | 12/06/2026 | Hoàn thành |

### Kết quả đạt được:
* **API Gateway + Orchestrator skeleton:** Endpoint `/recommendation` đã nhận request, parse userId/tier từ token Cognito và trả về response mẫu — chưa có logic quota/cache nhưng khung đã chạy được.
* **Schema DynamoDB hoàn chỉnh:** Hai bảng Users/Subscriptions và RecommendationCache được thiết kế chi tiết, sẵn sàng cho tuần sau triển khai logic nghiệp vụ.