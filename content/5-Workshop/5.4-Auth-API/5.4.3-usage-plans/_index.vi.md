---
title: "Cấu hình Usage Plans"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### Mục tiêu

Thêm **API Gateway Usage Plans** làm lớp giới hạn **kỹ thuật** (throttle + quota) để giảm spam và quá tải ngoài ý muốn.

{{% notice important %}}
Usage Plan **không** thay thế quota nghiệp vụ Free / Plus / Pro.

| Lớp | Nơi xử lý | Kiểm soát gì |
|---|---|---|
| **Usage Plan** | API Gateway | Request/giây hoặc /ngày theo API key (kỹ thuật) |
| **Quota gói dịch vụ** | DynamoDB + Lambda Orchestrator (Bước 5.5–5.6) | Tính năng theo gói, số lượt lịch trình/ngày, Verified Places Pro |

Quy tắc nghiệp vụ (nâng cấp gói, lượt còn lại) vẫn phải check trong **Lambda**, vì phụ thuộc trạng thái user realtime trên DynamoDB — không chỉ dựa vào API key.
{{% /notice %}}

#### Bước 1 — Tạo API keys (tùy chọn nhưng phổ biến)

1. API Gateway → **API keys** → **Create API key**
2. Tạo 3 key cho lab (tên — giá trị là secret):

| Tên key | Mapping lab |
|---|---|
| `wakan-free-key` | Giới hạn kỹ thuật gói Free |
| `wakan-plus-key` | Giới hạn kỹ thuật gói Plus |
| `wakan-pro-key` | Giới hạn kỹ thuật gói Pro |

<!-- TODO: screenshot - API keys list -->


{{% notice note %}}
Với SPA thuần JWT + Cognito, nhiều team **bỏ API key theo user** và chỉ dùng throttle mặc định trên stage. API key vẫn hữu ích cho demo, partner, hoặc tách môi trường. Nếu bỏ key, vẫn nên set **default stage throttling** trên stage `prod`.
{{% /notice %}}

#### Bước 2 — Tạo Usage Plans

1. API Gateway → **Usage plans** → **Create**
2. Tạo 3 plan:

| Tên plan | Throttle (rate / burst) | Quota (mỗi ngày) | Stage gắn |
|---|---|---|---|
| `wakan-plan-free` | 2 req/s · burst 5 | 100 requests/ngày | `wakan-api / prod` |
| `wakan-plan-plus` | 5 req/s · burst 10 | 500 requests/ngày | `wakan-api / prod` |
| `wakan-plan-pro` | 10 req/s · burst 20 | 2000 requests/ngày | `wakan-api / prod` |


#### Bước 3 — Bắt buộc API key trên method (chỉ khi dùng keys)

1. Mở `POST /itinerary` → **Method Request**
2. **API Key Required:** `true`
3. **Deploy API** lại lên stage `prod`

Client phải gửi:

```http
x-api-key: <your-api-key-value>
Authorization: <jwt>
```

#### Bước 4 — Throttle cấp stage (khuyến nghị kể cả không dùng key)

1. API Gateway → Stages → `prod` → **Settings** / **Default method throttling**
2. Đặt mức an toàn, ví dụ:
   - Rate: `5` requests/second
   - Burst: `10`

<!-- TODO: screenshot - stage default throttling -->
![Stage throttling](/worlog/images/5-Workshop/5.4-Auth-API/usage-stage-throttle.png)

#### Mapping với các gói Wakan

```
Request
  │
  ├─► Cognito Authorizer     → User là ai? (identity)
  ├─► Usage Plan / throttle  → Client có đang spam? (kỹ thuật)
  └─► Lambda Orchestrator    → Quota & tính năng Free/Plus/Pro (nghiệp vụ)
```

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Usage plans | Có Free / Plus / Pro (hoặc chỉ stage throttle) |
| Stage | `prod` đã gắn plan |
| API keys (nếu dùng) | Đã tạo và link vào plan |
| Test vượt limit | Vượt throttle → `429 Too Many Requests` |
