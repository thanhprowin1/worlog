---
title: "Test end-to-end"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.6.2. </b> "
---

#### Mục tiêu

Kiểm tra toàn bộ luồng:

> Login → WAF → CloudFront/s3 → Cognito Authorizer(login, JWT) → Frontend gửi API + token → WAF → API Gateway → Lambda (verify JWT / check admin / call Gemini) → DynamoDB → Response về frontend

#### Checklist trước khi test

| Kiểm tra | Trạng thái |
|---|---|
| Cognito test user đăng nhập được | ☐ |
| Item trong `wakan-users` khớp `sub` | ☐ |
| `wakan-cache` đã bật TTL | ☐ |
| Secret `wakan/ai-api-key` đã lưu | ☐ |
| Cả 2 Lambda đã deploy | ☐ |
| API Gateway integration = Lambda Orchestrator | ☐ |
| CloudFront `/api/*` forward `Authorization` | ☐ |

#### Test 1 — Unauthorized (không token)

```bash
curl -i -X POST \
  "https://dxxxx.cloudfront.net/api/itinerary" \
  -H "Content-Type: application/json" \
  -d '{
    "lat": 10.78,
    "lng": 106.70,
    "days": 1,
    "budget": "low",
    "companions": "solo",
    "transport": "motorbike",
    "experienceType": "food",
    "distanceScope": "near"
  }'
```

**Kỳ vọng:** `401 Unauthorized`

<!-- TODO: screenshot - curl 401 -->


#### Test 2 — Request đầu (cache MISS → AI)

1. Đăng nhập Hosted UI / frontend → copy Access token (hoặc ID token — thống nhất với authorizer)
2. Gọi:

```bash
export TOKEN="paste-jwt-here"
export CF="https://dxxxx.cloudfront.net"

curl -i -X POST \
  "$CF/api/itinerary" \
  -H "Content-Type: application/json" \
  -H "Authorization: $TOKEN" \
  -d '{
    "lat": 10.78,
    "lng": 106.70,
    "days": 1,
    "budget": "low",
    "companions": "solo",
    "transport": "motorbike",
    "experienceType": "food",
    "distanceScope": "near"
  }'
```

**Kỳ vọng:**
- HTTP `200`
- Body có `"source": "ai"` (hoặc itinerary từ AI lần đầu)
- Có thể mất vài giây (latency AI)

<!-- TODO: screenshot - first response source ai -->




#### Test 3 — Hết quota

1. Set `remainingQuota` = `0` cho test user trong DynamoDB
2. Gọi API lại với token hợp lệ

**Kỳ vọng:** `429` kèm message hết quota ngày

<!-- TODO: screenshot - 429 quota exceeded -->
![E2E 429](/worlog/images/5-Workshop/5.6-Business-logic/e2e-429.png)

#### Test 4 — CloudWatch logs

1. Mở **CloudWatch** → **Log groups**
2. Kiểm tra:
   - `/aws/lambda/wakan-orchestrator`
   - `/aws/lambda/wakan-ai-processor`
3. Xác nhận **không** có giá trị secret trong log

<!-- TODO: screenshot - CloudWatch log streams without secrets -->
![CloudWatch logs](/worlog/images/5-Workshop/5.6-Business-logic/e2e-logs.png)

#### Troubleshooting

| Triệu chứng | Nguyên nhân thường gặp |
|---|---|
| Luôn 401 | CloudFront không forward Authorization; sai loại token |
| 502 / timeout | Timeout AI Processor quá thấp; AI provider chậm/chặn |
| AccessDenied DynamoDB | IAM role thiếu quyền table |
| AccessDenied secret | Sai ARN / tên secret trong role |
| Itinerary rỗng | AI trả non-JSON; fallback chưa bắt lỗi parse |
| Cache không hit | `cacheKey` khác nhẹ (float, thiếu field option) |

#### Checkpoint — hoàn thành Bước 5.6

| Mục | Kỳ vọng |
|---|---|
| Mock đã thay | API Gateway → Lambda Orchestrator |
| Hai Lambda | Orchestrator + AI Processor |
| Secrets | API key chỉ nằm trong Secrets Manager |
| Cache chạy | Request 2 giống hệt nhanh hơn / `source: cache` |
| Quota chạy | 429 khi remainingQuota = 0 |
| Log sạch | Không lộ API key trên CloudWatch |

Tiếp theo: **Bước 5.7** đi sâu bảo mật, metrics giám sát, và AWS Budgets kiểm soát chi phí.
