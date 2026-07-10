---
title: "Xử lý nghiệp vụ với Lambda"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Mục tiêu

Trong phần này, bạn sẽ tạo và kết nối 2 hàm Lambda tạo nên lõi nghiệp vụ của Wakan:

| Hàm | Trách nhiệm |
|---|---|
| **Orchestrator** | Validate đầu vào, kiểm tra gói + quota từ DynamoDB, tạo cache key, trả kết quả cache nếu có |
| **AI Processor** | Lấy API key từ Secrets Manager, xây dựng prompt có cấu trúc, gọi mô hình AI bên ngoài, ép định dạng JSON output, xử lý fallback |

```
Frontend POST /api/itinerary
    │
    ▼
API Gateway ──► Cognito Authorizer
    │
    ▼
Lambda Orchestrator
    ├── GetItem(wakan-users)     → plan + remainingQuota
    ├── GetItem(wakan-cache)     → cache HIT? trả ngay
    │   └── MISS?
    │       │
    │       ▼
    │   Lambda AI Processor
    │       ├── GetSecretValue(Secrets Manager)
    │       ├── Gọi AI API (xây prompt từ tùy chọn đã validate)
    │       ├── Validate JSON schema output
    │       ├── PutItem(wakan-cache)   → lưu để dùng lại sau
    │       └── Trả về itinerary
    │
    ▼
Orchestrator trừ remainingQuota trong wakan-users
    │
    ▼
Response về frontend
```

{{% notice note %}}
Bước này thay thế **MOCK** integration từ Bước 5.4 bằng Lambda thật. Cognito Authorizer giữ nguyên.
{{% /notice %}}

#### Nội dung

1. [Tạo Lambda AI Processor](5.6.1-ai-processor/)
2. [Test end-to-end](5.6.2-e2e-test/)
