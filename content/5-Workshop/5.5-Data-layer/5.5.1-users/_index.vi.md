---
title: "Tạo bảng Users / Subscriptions"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

#### Mục tiêu

Tạo bảng lưu gói dịch vụ và quota còn lại của từng user. Quota **không** chỉ nằm trong JWT — Lambda luôn đọc trạng thái mới nhất từ DynamoDB để nâng cấp gói có hiệu lực ngay.

#### Thiết kế bảng

| Thuộc tính | Giá trị |
|---|---|
| **Tên bảng** | `wakan-users` |
| **Partition key (PK)** | `userId` (String) — dùng Cognito `sub` |
| **Sort key** | không (MVP: 1 item / user) |
| **Capacity** | On-demand (PAY_PER_REQUEST) |

**Attribute gợi ý (DynamoDB không bắt buộc khai báo hết):**

| Attribute | Type | Ví dụ | Ghi chú |
|---|---|---|---|
| `userId` | S | Cognito `sub` | Primary key |
| `email` | S | `user@example.com` | Tiện tra cứu |
| `plan` | S | `free` \| `plus` \| `pro` | Gói hiện tại |
| `dailyQuota` | N | `3` | Tối đa lượt/ngày theo gói |
| `remainingQuota` | N | `2` | Orchestrator trừ dần |
| `quotaResetAt` | S | `2026-07-10T00:00:00Z` | Mốc reset quota (UTC) |
| `createdAt` | S | ISO-8601 | |
| `updatedAt` | S | ISO-8601 | |

#### Quota mặc định theo gói (lab)

| Gói | `dailyQuota` | Ghi chú |
|---|---|---|
| Free | `3` | Chỉ gần / ngắn ngày (enforce ở Lambda) |
| Plus | `10` | Cho phép xa + nhiều ngày |
| Pro | `999` hoặc số lớn | Lab coi như unlimited |

#### Bước 1 — Tạo bảng

1. Mở **DynamoDB** → **Tables** → **Create table**
2. Cấu hình:
   - **Table name:** `TripAITravelPlan`
   - **Partition key:** `userId` · type **String**
   - **Capacity mode:** **On-demand**
   - Encryption: AWS owned key (mặc định)
3. Bấm **Create table**

<!-- TODO: screenshot - form Create table wakan-users -->
![Tạo bảng wakan-users](/worlog/images/5-Workshop/5.5-Data-layer/users-create.png)

4. Đợi status **Active**

<!-- TODO: screenshot - table Active trong danh sách -->
![wakan-users Active](/worlog/images/5-Workshop/5.5-Data-layer/users-active1.png)

#### Bước 2 — Chèn user mẫu (test)

1. Mở bảng `TripAITravelPlan` → **Explore table items** → **Create item**
2. Dùng JSON view, dán (thay `userId` bằng **sub** của Cognito test user):

```json
{
  "userId": {
    "S": "PASTE-COGNITO-SUB-HERE"
  },
  "email": {
    "S": "test@example.com"
  },
  "plan": {
    "S": "free"
  },
  "dailyQuota": {
    "N": "3"
  },
  "remainingQuota": {
    "N": "3"
  },
  "quotaResetAt": {
    "S": "2026-07-10T00:00:00Z"
  },
  "createdAt": {
    "S": "2026-07-09T12:00:00Z"
  },
  "updatedAt": {
    "S": "2026-07-09T12:00:00Z"
  }
}
```

3. Bấm **Create item**

<!-- TODO: screenshot - item Free user trong table -->
![Item user Free mẫu](/worlog/images/5-Workshop/5.5-Data-layer/users-sample-item.png)

**Lấy Cognito `sub`:**

- Sau login Hosted UI, decode JWT (jwt.io) → claim `sub`, **hoặc**
- Cognito → Users → mở user → **User sub** / attributes

#### Bước 3 — (Tùy chọn) item Plus / Pro

Tạo item khác cùng cấu trúc, ví dụ:

```json
"plan": { "S": "pro" },
"dailyQuota": { "N": "999" },
"remainingQuota": { "N": "999" }
```

Hữu ích khi test Pro + Verified Places.

#### CLI thay thế

```bash
aws dynamodb create-table \
  --table-name wakan-users \
  --attribute-definitions AttributeName=userId,AttributeType=S \
  --key-schema AttributeName=userId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-southeast-1
```

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Bảng | `wakan-users` status **Active** |
| PK | `userId` (String) |
| Capacity | On-demand |
| Item mẫu | Ít nhất 1 user Free có `remainingQuota` |
| Liên kết Cognito | `userId` = `sub` thật của test user |
