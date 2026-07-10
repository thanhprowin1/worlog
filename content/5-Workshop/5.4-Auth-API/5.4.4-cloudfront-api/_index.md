---
title: "Route /api/* through CloudFront"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### Objective

Add API Gateway as a **second origin** on your CloudFront distribution and create a cache behavior for `/api/*`, so the browser only talks to **one domain** (the CloudFront domain).

```
https://dxxxx.cloudfront.net/           →  S3 (frontend)
https://dxxxx.cloudfront.net/api/*      →  API Gateway
```

#### Step 1 — Prepare API Gateway origin domain

From API Gateway stage `prod`, copy the **Invoke URL** host only, for example:

```
xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com
```

You will **not** put `/prod` into the origin domain field — stage path is handled via origin path or full path forwarding (see below).

#### Step 2 — Add API Gateway origin

1. Open **CloudFront** → your distribution → **Origins** → **Create origin**
2. Configure:
   - **Origin domain:** `xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com`
   - **Protocol:** HTTPS only
   - **Origin path:** `/prod`  
     *(so a request to CloudFront `/api/itinerary` can be mapped carefully — see note below)*
   - **Name:** `wakan-api-origin`
   - **Add custom header** (optional hardening): e.g. `X-Origin-Verify: <random-secret>`  
     *(only useful if you later validate it in API Gateway / Lambda)*

<!-- TODO: screenshot - Create origin API Gateway -->
![API Gateway origin](/images/5-Workshop/5.4-Auth-API/cf-api-origin.png)

{{% notice note %}}
**Path mapping (lab recommendation)**

Simplest reliable setup for this workshop:

1. API Gateway method remains `POST /itinerary`
2. CloudFront behavior path pattern: `/api/*`
3. Origin path: `/prod`
4. Frontend calls: `POST https://dxxxx.cloudfront.net/api/itinerary`

CloudFront forwards `/api/itinerary` to origin. With **origin path `/prod`**, the origin request path becomes `/prod/api/itinerary` unless you strip the `/api` prefix.

**Two practical options:**

| Option | API Gateway resource | Frontend path | Origin path | Notes |
|---|---|---|---|---|
| **A (recommended)** | Create resource `/api/itinerary` (or nested `api` → `itinerary`) | `/api/itinerary` | `/prod` | Path matches end-to-end |
| **B** | Keep `/itinerary` | `/api/itinerary` | empty + use CloudFront Function to rewrite `/api/*` → `/*` then origin path `/prod` | Cleaner URLs, slightly more setup |

**Use Option A for the lab** unless you already know CloudFront Functions.
{{% /notice %}}

#### Step 2b — Align API resource with Option A (if needed)

If you created only `/itinerary` in 5.4.2:

1. Create parent resource `api`
2. Create child `itinerary` under `api` → path `/api/itinerary`
3. Move or recreate `POST` with Cognito authorizer + mock (same as before)
4. Enable CORS if needed → **Deploy** to `prod` again

<!-- TODO: screenshot - resource tree /api/itinerary -->
![API resource /api/itinerary](/images/5-Workshop/5.4-Auth-API/apigw-api-resource.png)

#### Step 3 — Create cache behavior for `/api/*`

1. CloudFront distribution → **Behaviors** → **Create behavior**
2. Configure:

| Setting | Value |
|---|---|
| **Path pattern** | `/api/*` |
| **Origin** | `wakan-api-origin` |
| **Viewer protocol policy** | Redirect HTTP to HTTPS |
| **Allowed HTTP methods** | GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE |
| **Cache policy** | **CachingDisabled** |
| **Origin request policy** | **AllViewerExceptHostHeader** (or a policy that forwards `Authorization` and `Content-Type`) |
| **Response headers policy** | Optional (CORS if required) |

{{% notice important %}}
You **must forward the `Authorization` header** to API Gateway. If CloudFront strips it, Cognito Authorizer will always return 401.
{{% /notice %}}

<!-- TODO: screenshot - Create behavior /api/* settings -->
![Behavior /api/*](/images/5-Workshop/5.4-Auth-API/cf-api-behavior.png)

3. Create behavior. Ensure precedence: `/api/*` is **more specific** than Default (`*`) — CloudFront evaluates specific patterns first.

#### Step 4 — Wait for deployment & test

1. Wait until distribution status is **Deployed**
2. Test without token:

```bash
curl -i -X POST \
  "https://dxxxx.cloudfront.net/api/itinerary" \
  -H "Content-Type: application/json" \
  -d '{"budget":"low","days":1}'
```

Expect **401**.

3. Test with token:

```bash
curl -i -X POST \
  "https://dxxxx.cloudfront.net/api/itinerary" \
  -H "Content-Type: application/json" \
  -H "Authorization: $TOKEN" \
  -d '{"budget":"low","days":1,"companions":"solo"}'
```

Expect **200** + mock JSON from Step 5.4.2.

<!-- TODO: screenshot - curl qua CloudFront domain 401 + 200 -->
![Test via CloudFront](/images/5-Workshop/5.4-Auth-API/cf-api-test.png)

#### Final edge map

| Path | Origin | Cache | Auth |
|---|---|---|---|
| Default `*` | S3 (OAC) | Cached | Public static files |
| `/api/*` | API Gateway | **No cache** | Cognito JWT required |

#### Checkpoint

| Item | Expected result |
|---|---|
| Second origin | API Gateway origin on the distribution |
| Behavior | Path `/api/*` → API origin, CachingDisabled |
| Authorization header | Forwarded to origin |
| `POST /api/itinerary` no token | 401 |
| `POST /api/itinerary` with token | 200 mock body |
| Frontend static pages | Still load on `/` |
