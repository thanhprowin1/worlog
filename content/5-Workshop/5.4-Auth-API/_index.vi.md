---
title: "Xác thực & lớp API"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Mục tiêu

Trong phần này, bạn sẽ dựng lớp xác thực và cổng API của Wakan:

- **Amazon Cognito** cho đăng ký, đăng nhập và JWT token
- **Amazon API Gateway** làm REST API front door
- **Cognito Authorizer** để chỉ user đã đăng nhập mới gọi được route bảo vệ
- **Usage Plans** giới hạn request ở mức kỹ thuật (chống spam)
- **CloudFront behavior `/api/*`** để frontend và API dùng chung một domain

```
Trình duyệt
  │  đăng nhập
  ▼
Cognito Hosted UI  ──►  ID / Access token (JWT)
  │
  │  Authorization: Bearer <token>
  ▼
CloudFront  (/api/*)
  │
  ▼
API Gateway  ──► Cognito Authorizer  ──►  (Lambda ở Bước 5.6)
```

{{% notice note %}}
Tích hợp Lambda sẽ hoàn tất ở **Bước 5.6**. Trong phần này bạn tạo Cognito, API Gateway, authorizer, routes, và (tùy chọn) **MOCK** integration để kiểm tra JWT đã validate đúng.
{{% /notice %}}

#### Nội dung

1. [Tạo Cognito User Pool](5.4.1-cognito/)
2. [Tạo API Gateway & Cognito Authorizer](5.4.2-api-gateway/)
3. [Cấu hình Usage Plans](5.4.3-usage-plans/)
4. [Chuyển `/api/*` qua CloudFront](5.4.4-cloudfront-api/)
