---
title: "Chuyển /api/* qua CloudFront"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### Mục tiêu

Thêm API Gateway làm **origin thứ hai** trên CloudFront distribution và tạo cache behavior cho `/api/*`, để trình duyệt chỉ nói chuyện với **một domain** (CloudFront).

```
https://dxxxx.cloudfront.net/           →  S3 (frontend)
https://dxxxx.cloudfront.net/api/*      →  API Gateway
```

#### Bước 1 — Chuẩn bị domain origin API Gateway

Từ stage `prod` của API Gateway, copy **host** của Invoke URL, ví dụ:

```
xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com
```

**Không** ghi `/prod` vào trường origin domain — stage path xử lý qua origin path / mapping (xem bên dưới).

#### Bước 2 — Thêm origin API Gateway

1. Mở **CloudFront** → distribution → **Origins** → **Create origin**
2. Cấu hình:
   - **Origin domain:** `xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com`
   - **Protocol:** HTTPS only
   - **Origin path:** `/prod`
   - **Name:** `wakan-api-origin`
   - **Add custom header** (tùy chọn): `X-Origin-Verify: <random-secret>`  
     *(chỉ hữu ích nếu sau này validate ở API Gateway / Lambda)*

<!-- TODO: screenshot - Create origin API Gateway -->
![API Gateway origin](/worlog/images/5-Workshop/5.4-Auth-API/cf-api-origin.png)

{{% notice note %}}
**Path mapping (khuyến nghị lab)**

Cách đơn giản, ổn định:

1. Method API Gateway: `POST /api/itinerary` (resource lồng `api` → `itinerary`)
2. CloudFront behavior: `/api/*`
3. Origin path: `/prod`
4. Frontend gọi: `POST https://dxxxx.cloudfront.net/api/itinerary`

**Hai lựa chọn:**

| Option | Resource API Gateway | Path frontend | Origin path | Ghi chú |
|---|---|---|---|---|
| **A (khuyến nghị)** | `/api/itinerary` | `/api/itinerary` | `/prod` | Path khớp end-to-end |
| **B** | `/itinerary` | `/api/itinerary` | rewrite bằng CloudFront Function | Gọn URL, setup phức tạp hơn |

**Lab dùng Option A** trừ khi bạn đã quen CloudFront Functions.
{{% /notice %}}

#### Bước 2b — Chỉnh resource API theo Option A (nếu cần)

Nếu ở 5.4.2 bạn mới tạo `/itinerary`:

1. Tạo parent resource `api`
2. Tạo child `itinerary` dưới `api` → path `/api/itinerary`
3. Tạo lại `POST` với Cognito authorizer + mock (như trước)
4. Bật CORS nếu cần → **Deploy** lại `prod`

<!-- TODO: screenshot - resource tree /api/itinerary -->
![API resource /api/itinerary](/worlog/images/5-Workshop/5.4-Auth-API/apigw-api-resource.png)

#### Bước 3 — Tạo cache behavior `/api/*`

1. CloudFront distribution → **Behaviors** → **Create behavior**
2. Cấu hình:

| Setting | Giá trị |
|---|---|
| **Path pattern** | `/api/*` |
| **Origin** | `wakan-api-origin` |
| **Viewer protocol policy** | Redirect HTTP to HTTPS |
| **Allowed HTTP methods** | GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE |
| **Cache policy** | **CachingDisabled** |
| **Origin request policy** | **AllViewerExceptHostHeader** (hoặc policy forward `Authorization` + `Content-Type`) |
| **Response headers policy** | Tùy chọn (CORS nếu cần) |

{{% notice important %}}
Bạn **phải forward header `Authorization`** tới API Gateway. Nếu CloudFront cắt header này, Cognito Authorizer sẽ luôn trả 401.
{{% /notice %}}

<!-- TODO: screenshot - Create behavior /api/* settings -->


3. Create behavior. Pattern `/api/*` cụ thể hơn Default (`*`) — CloudFront ưu tiên pattern cụ thể.

#### Bước 4 — Chờ deploy & test

1. Đợi distribution **Deployed**
2. Test không token:

```bash
curl -i -X POST \
  "https://dxxxx.cloudfront.net/api/itinerary" \
  -H "Content-Type: application/json" \
  -d '{"budget":"low","days":1}'
```

Kỳ vọng **401**.

3. Test có token:

```bash
curl -i -X POST \
  "https://dxxxx.cloudfront.net/api/itinerary" \
  -H "Content-Type: application/json" \
  -H "Authorization: $TOKEN" \
  -d '{"budget":"low","days":1,"companions":"solo"}'
```

Kỳ vọng **200** + mock JSON từ Bước 5.4.2.

<!-- TODO: screenshot - curl qua CloudFront domain 401 + 200 -->


#### Bản đồ biên cuối phần 5.4

| Path | Origin | Cache | Auth |
|---|---|---|---|
| Default `*` | S3 (OAC) | Có cache | File tĩnh public |
| `/api/*` | API Gateway | **Không cache** | Bắt buộc Cognito JWT |

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Origin thứ hai | API Gateway trên distribution |
| Behavior | Path `/api/*` → API origin, CachingDisabled |
| Header Authorization | Được forward tới origin |
| `POST /api/itinerary` không token | 401 |
| `POST /api/itinerary` có token | 200 mock body |
| Trang tĩnh frontend | Vẫn mở được `/` |
