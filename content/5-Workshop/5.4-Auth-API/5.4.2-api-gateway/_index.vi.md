---
title: "Tạo API Gateway & Cognito Authorizer"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### Mục tiêu

Tạo REST API có:

1. Endpoint chính Wakan `POST /itinerary` (public path `/api/itinerary` qua CloudFront)
2. Validate JWT bằng **Cognito User Pool authorizer**
3. Integration tạm **MOCK** để test auth trước khi gắn Lambda (Bước 5.6)

#### Bước 1 — Tạo REST API

1. Mở **API Gateway** → **APIs** → **Create API** → **REST API** → **Build**
2. Cấu hình:
   - **API name:** `TripAI-API`
   - **Endpoint type:** Regional
3. Bấm **Create API**

<!-- TODO: screenshot - Create REST API -->
![Tạo REST API](/worlog/images/5-Workshop/5.4-Auth-API/apigw-create.png)

#### Bước 2 — Tạo Cognito authorizer

1. Trong `wakan-api` → **Authorizers** → **Create authorizer**
2. Cấu hình:
   - **Name:** `wakan-cognito-authorizer`
   - **Type:** Cognito
   - **Cognito user pool:** chọn `User pool - weu7cv`
   - **Token source:** `Authorization`
   - **Token validation** (tùy chọn): để trống trong lab
3. Bấm **Create authorizer**
4. **Test** (tùy chọn): dán Access token từ Hosted UI → phải trả claims (`sub`, `email`, …)

<!-- TODO: screenshot - Cognito authorizer created + Test thành công -->


{{% notice note %}}
**Access token vs ID token:** Authorizer User Pool thường cần token khớp issuer/audience. Nếu Test fail với Access token, thử **ID token** từ cùng lần login, hoặc chỉnh scopes app client. Chọn một loại và dùng thống nhất trên frontend.
{{% /notice %}}

#### Bước 3 — Tạo resource & method

Cấu trúc resource khuyến nghị:

```
/
└── itinerary     POST   (protected)
```

**Mặc định lab (đơn giản):**

1. **Create resource**
   - Resource name: `itinerary`
   - Resource path: `/itinerary`
2. **Create method** `POST` trên `/itinerary`
3. Integration type (tạm): **Mock**
4. Method Request → **Authorization:** chọn `wakan-cognito-authorizer`
5. Integration Response → body JSON mẫu:

```json
{
  "message": "Wakan API is up. Lambda will be connected in Step 5.6.",
  "ok": true
}
```

6. Method Response → thêm `200` với content-type `application/json`

<!-- TODO: screenshot - resource /itinerary + POST method + authorizer attached -->


#### Bước 4 — Bật CORS (nếu gọi thẳng domain API)

Nếu browser gọi **URL API Gateway** trực tiếp (không chỉ qua CloudFront same origin), bật CORS:

1. Chọn `/itinerary` → **Enable CORS**
2. Allow methods: `POST, OPTIONS`
3. Allow headers: `Authorization, Content-Type`
4. Allow origin: CloudFront của bạn `https://dxxxx.cloudfront.net` (hoặc `*` chỉ cho lab)

<!-- TODO: screenshot - Enable CORS -->

{{% notice tip %}}
Khi frontend và API cùng domain CloudFront, same-origin giúp giảm nhu cầu CORS. Vẫn hữu ích khi test frontend local.
{{% /notice %}}

#### Bước 5 — Deploy API

1. **Deploy API**
2. **Stage name:** `prod` (hoặc `dev`)
3. Ghi lại **Invoke URL**, ví dụ:  
   `https://xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod`

<!-- TODO: screenshot - stage prod + Invoke URL -->


#### Bước 6 — Test bằng curl

Không có token → **401 / Unauthorized**:

```bash
curl -i -X POST \
  "https://xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/itinerary" \
  -H "Content-Type: application/json" \
  -d '{"budget":"low","days":1}'
```

Có token → **200** + mock JSON:

```bash
export TOKEN="paste-jwt-here"

curl -i -X POST \
  "https://xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/itinerary" \
  -H "Content-Type: application/json" \
  -H "Authorization: $TOKEN" \
  -d '{"budget":"low","days":1,"companions":"solo"}'
```

<!-- TODO: screenshot - terminal 401 without token + 200 with token -->


#### Sẽ thay đổi gì ở Bước 5.6

| Hiện tại (5.4) | Sau này (5.6) |
|---|---|
| Integration = **Mock** | Integration = **Lambda** (Orchestrator) |
| JSON mock cố định | Lịch trình thật từ cache / AI |
| Authorizer đã gắn | Giữ nguyên authorizer |

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| REST API | Tên `TripAI-API`, stage `prod` |
| Authorizer | Cognito authorizer gắn `wakan-users` |
| Method | `POST /itinerary` bắt buộc Authorization |
| Không token | 401 Unauthorized |
| Token hợp lệ | 200 + mock response |
