---
title: "Lớp dữ liệu với DynamoDB"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Mục tiêu

Trong phần này, bạn sẽ tạo các bảng DynamoDB lưu dữ liệu cốt lõi của Wakan:

| Bảng | Mục đích |
|---|---|
| **Users / Subscriptions** | Liên kết identity user, gói (Free / Plus / Pro), quota còn lại trong ngày |
| **Cache** | Lịch trình đã tạo, key theo tọa độ làm tròn + tùy chọn, có TTL |
| **Verified Places** | Địa điểm đã kiểm chứng cho gói **Pro** để AI không tự “bịa” địa điểm |

```
Lambda Orchestrator
    │
    ├── GetItem / UpdateItem  →  wakan-users
    ├── GetItem / PutItem     →  wakan-cache   (TTL)
    └── Query / Scan (Pro)    →  wakan-verified-places
```

{{% notice note %}}
Code Lambda đọc/ghi các bảng này sẽ làm ở **Bước 5.6**. Ở đây bạn chỉ tạo bảng, định nghĩa key, bật TTL, và chèn dữ liệu mẫu để test.
{{% /notice %}}

#### Nội dung

1. [Tạo bảng Users / Subscriptions](5.5.1-users/)
2. [Tạo bảng Cache với TTL](5.5.2-cache/)

