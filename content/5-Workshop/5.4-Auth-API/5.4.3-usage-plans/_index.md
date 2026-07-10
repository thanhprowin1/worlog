---
title: "Configure Usage Plans"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### Objective

Add **API Gateway Usage Plans** as a **technical** rate-limit layer (throttle + quota) to reduce spam and accidental overload.

{{% notice important %}}
Usage Plans are **not** a substitute for Free / Plus / Pro **business quotas**.

| Layer | Where | What it controls |
|---|---|---|
| **Usage Plan** | API Gateway | Requests per second / day per API key (technical) |
| **Subscription quota** | DynamoDB + Lambda Orchestrator (Step 5.5–5.6) | Plan features, daily itinerary limits, Pro verified places |

Business rules (upgrade plan, remaining daily credits) must still be enforced in **Lambda**, because they depend on live user state in DynamoDB — not only on an API key.
{{% /notice %}}

#### Step 1 — Create API keys (optional but common)

1. API Gateway → **API keys** → **Create API key**
2. Create three keys for the lab (names only — values are secrets):

| Key name | Intended mapping (lab) |
|---|---|
| `wakan-free-key` | Free tier technical limit |
| `wakan-plus-key` | Plus tier technical limit |
| `wakan-pro-key` | Pro tier technical limit |

<!-- TODO: screenshot - API keys list -->


{{% notice note %}}
For a pure JWT + Cognito SPA, many teams **skip per-user API keys** and only use Usage Plan default stage throttling. API keys are still useful for demos, partner access, or separating environments. If you skip keys, still set **default stage throttling** on the `prod` stage.
{{% /notice %}}

#### Step 2 — Create Usage Plans

1. API Gateway → **Usage plans** → **Create**
2. Create three plans:

| Plan name | Throttle (rate / burst) | Quota (per day) | Associated stage |
|---|---|---|---|
| `wakan-plan-free` | 2 req/s · burst 5 | 100 requests/day | `wakan-api / prod` |
| `wakan-plan-plus` | 5 req/s · burst 10 | 500 requests/day | `wakan-api / prod` |
| `wakan-plan-pro` | 10 req/s · burst 20 | 2000 requests/day | `wakan-api / prod` |


#### Step 3 — Require API key on method (only if using keys)

1. Open `POST /itinerary` → **Method Request**
2. **API Key Required:** `true`
3. **Deploy API** again to stage `prod`

Clients must send:

```http
x-api-key: <your-api-key-value>
Authorization: <jwt>
```

#### Step 4 — Stage-level throttle (recommended even without keys)

1. API Gateway → Stages → `prod` → **Settings** / **Default method throttling**
2. Set a conservative default, e.g.:
   - Rate: `5` requests/second
   - Burst: `10`

<!-- TODO: screenshot - stage default throttling -->
![Stage throttling](/images/5-Workshop/5.4-Auth-API/usage-stage-throttle.png)

#### How this maps to Wakan tiers

```
Request
  │
  ├─► Cognito Authorizer     → Who is the user? (identity)
  ├─► Usage Plan / throttle  → Is this client spamming? (technical)
  └─► Lambda Orchestrator    → Free/Plus/Pro quota & features (business)
```

#### Checkpoint

| Item | Expected result |
|---|---|
| Usage plans | Free / Plus / Pro plans exist (or stage throttle only) |
| Stage | `prod` associated with plan(s) |
| Optional API keys | Created and linked if you chose that model |
| Over-limit test | Exceed throttle → `429 Too Many Requests` |
