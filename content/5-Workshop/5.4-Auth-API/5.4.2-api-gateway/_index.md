---
title: "Create API Gateway & Cognito Authorizer"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### Objective

Create a REST API that:

1. Exposes the main Wakan endpoint `POST /api/itinerary` (resource path can be `/itinerary` under stage/base path)
2. Validates JWT via a **Cognito User Pool authorizer**
3. Uses a temporary **MOCK** integration so you can test auth before Lambda is ready (Step 5.6)

#### Step 1 — Create REST API

1. Open **API Gateway** → **APIs** → **Create API** → **REST API** → **Build**
2. Configure:
   - **API name:** `TripAI-API`
   - **Endpoint type:** Regional
3. Click **Create API**

<!-- TODO: screenshot - Create REST API -->
![Create REST API](/worlog/images/5-Workshop/5.4-Auth-API/apigw-create.png)

#### Step 2 — Create Cognito authorizer

1. In `wakan-api` → **Authorizers** → **Create authorizer**
2. Configure:
   - **Name:** `wakan-cognito-authorizer`
   - **Type:** Cognito
   - **Cognito user pool:** select `User pool - weu7cv`
   - **Token source:** `Authorization`
   - **Token validation** (optional): leave empty for lab
3. Click **Create authorizer**
4. **Test** (optional): paste an Access token from Hosted UI login → should return claims (`sub`, `email`, …)

<!-- TODO: screenshot - Cognito authorizer created + Test thành công -->


{{% notice note %}}
**Access token vs ID token:** Cognito User Pool authorizer typically expects a token whose `aud` / issuer matches the user pool. If Test fails with Access token, try the **ID token** from the same login response, or configure the app client / scopes accordingly. Keep one choice and use it consistently in the frontend.
{{% /notice %}}

#### Step 3 — Create resources & method

Recommended resource structure:

```
/
└── itinerary     POST   (protected)
```

If you prefer the public path to be `/api/itinerary` **on CloudFront only**, keep API Gateway resource as `/itinerary` and map CloudFront path `/api/*` with origin path rewrite — or create resource `/api/itinerary` on API Gateway.  
**Simplest for lab:** resource path `/itinerary`, CloudFront origin path empty, behavior path pattern `/api/*` with origin request policy that forwards the full path **or** use API stage base path mapping.  

**Lab default (clear & simple):**

1. **Create resource**
   - Resource name: `itinerary`
   - Resource path: `/itinerary`
2. **Create method** `POST` on `/itinerary`
3. Integration type (temporary): **Mock**
4. Method Request → **Authorization:** select `wakan-cognito-authorizer`
5. Integration Response → return a simple JSON body, e.g.:

```json
{
  "message": "Wakan API is up. Lambda will be connected in Step 5.6.",
  "ok": true
}
```

6. Method Response → add `200` with response body model `Empty` or application/json

<!-- TODO: screenshot - resource /itinerary + POST method + authorizer attached -->


#### Step 4 — Enable CORS (if calling API domain directly)

If the browser ever calls the **API Gateway URL** directly (not only via CloudFront same origin), enable CORS:

1. Select `/itinerary` → **Enable CORS**
2. Allow methods: `POST, OPTIONS`
3. Allow headers: `Authorization, Content-Type`
4. Allow origin: your CloudFront origin `https://dxxxx.cloudfront.net` (or `*` for lab only)

<!-- TODO: screenshot - Enable CORS -->


{{% notice tip %}}
When frontend and API are both served from the **same CloudFront domain**, browser same-origin applies and CORS is less critical. Still useful for local frontend testing.
{{% /notice %}}

#### Step 5 — Deploy API

1. **Actions** / **Deploy API**
2. **Stage name:** `prod` (or `dev`)
3. Note the **Invoke URL**, e.g.  
   `https://xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod`

<!-- TODO: screenshot - stage prod + Invoke URL -->


#### Step 6 — Test with curl

Without token → should be **401 / Unauthorized**:

```bash
curl -i -X POST \
  "https://xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/itinerary" \
  -H "Content-Type: application/json" \
  -d '{"budget":"low","days":1}'
```

With token → **200** + mock JSON:

```bash
export TOKEN="paste-jwt-here"

curl -i -X POST \
  "https://xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/itinerary" \
  -H "Content-Type: application/json" \
  -H "Authorization: $TOKEN" \
  -d '{"budget":"low","days":1,"companions":"solo"}'
```

<!-- TODO: screenshot - terminal 401 without token + 200 with token -->


#### What you will change in Step 5.6

| Now (5.4) | Later (5.6) |
|---|---|
| Integration = **Mock** | Integration = **Lambda** (Orchestrator) |
| Fixed mock JSON | Real itinerary from cache / AI |
| Authorizer already on | Keep authorizer as-is |

#### Checkpoint

| Item | Expected result |
|---|---|
| REST API | Name `TripAI-API`, stage `prod` |
| Authorizer | Cognito authorizer linked to `wakan-users` |
| Method | `POST /itinerary` requires Authorization |
| No token | 401 Unauthorized |
| Valid token | 200 + mock response |
