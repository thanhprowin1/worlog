---
title: "End-to-end test"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.6.2. </b> "
---

#### Objective

Verify the full path:

> Login → WAF → CloudFront/s3 → Cognito Authorizer(login, JWT) → Frontend gửi API + token → WAF → API Gateway → Lambda (verify JWT / check admin / call Gemini) → DynamoDB → Response về frontend

#### Prerequisites checklist

| Check | Status |
|---|---|
| Cognito test user can sign in | ☐ |
| Sample item in `wakan-users` with matching `sub` | ☐ |
| `wakan-cache` TTL enabled | ☐ |
| Secret `wakan/ai-api-key` stored | ☐ |
| Both Lambdas deployed | ☐ |
| API Gateway integration = Lambda Orchestrator | ☐ |
| CloudFront `/api/*` behavior forwards `Authorization` | ☐ |

#### Test 1 — Unauthorized (no token)

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

**Expected:** `401 Unauthorized`

<!-- TODO: screenshot - curl 401 -->


#### Test 2 — First request (cache MISS → AI)

1. Sign in via Hosted UI / frontend → copy Access token (or ID token — same as authorizer config)
2. Call:

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

**Expected:**
- HTTP `200`
- Body contains `"source": "ai"` (or itinerary without cache flag on first call)
- Duration may be several seconds (AI latency)

<!-- TODO: screenshot - first response source ai -->




#### Test 3 — Quota exceeded

1. Set `remainingQuota` to `0` for the test user in DynamoDB
2. Call API again with valid token

**Expected:** `429` with message about daily quota

<!-- TODO: screenshot - 429 quota exceeded -->
![E2E 429](/worlog/images/5-Workshop/5.6-Business-logic/e2e-429.png)

#### Test 4 — CloudWatch logs

1. Open **CloudWatch** → **Log groups**
2. Check:
   - `/aws/lambda/wakan-orchestrator`
   - `/aws/lambda/wakan-ai-processor`
3. Confirm no secret values appear in logs

<!-- TODO: screenshot - CloudWatch log streams without secrets -->
![CloudWatch logs](/worlog/images/5-Workshop/5.6-Business-logic/e2e-logs.png)

#### Troubleshooting

| Symptom | Likely cause |
|---|---|
| Always 401 | Authorization header not forwarded by CloudFront; wrong token type |
| 502 / timeout | AI Processor timeout too low; AI provider slow/blocked |
| AccessDenied on DynamoDB | IAM role missing table permissions |
| AccessDenied on secret | Wrong secret ARN / name in role |
| Empty itinerary | AI returned non-JSON; fallback not handling parse error |
| Cache never hits | `cacheKey` fields differ slightly (float precision, missing option) |

#### Checkpoint — Step 5.6 complete

| Item | Expected |
|---|---|
| Mock replaced | API Gateway → Lambda Orchestrator |
| Two Lambdas | Orchestrator + AI Processor |
| Secrets | API key only in Secrets Manager |
| Cache works | Second identical request is faster / `source: cache` |
| Quota works | 429 when remainingQuota = 0 |
| Logs clean | No API keys in CloudWatch |

Next: **Step 5.7** deepens security, monitoring metrics, and AWS Budgets for cost control.
