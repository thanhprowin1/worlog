---
title: "Create Users / Subscriptions table"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

#### Objective

Create the table that stores each user’s plan and remaining daily quota. Quota is **not** stored only in the JWT — Lambda always reads the latest state from DynamoDB so upgrades take effect immediately.

#### Table design

| Property | Value |
|---|---|
| **Table name** | `TripAITravelPlan` |
| **Partition key (PK)** | `userId` (String) — use Cognito `sub` |
| **Sort key** | none (single-item per user for MVP) |
| **Capacity** | On-demand (PAY_PER_REQUEST) |

**Suggested attributes (no need to declare all in DynamoDB):**

| Attribute | Type | Example | Notes |
|---|---|---|---|
| `userId` | S | Cognito `sub` | Primary key |
| `email` | S | `user@example.com` | Optional convenience |
| `plan` | S | `free` \| `plus` \| `pro` | Current subscription |
| `dailyQuota` | N | `3` | Max itineraries per day for this plan |
| `remainingQuota` | N | `2` | Decremented by Orchestrator |
| `quotaResetAt` | S | `2026-07-10T00:00:00Z` | When remainingQuota resets (UTC day) |
| `createdAt` | S | ISO-8601 | |
| `updatedAt` | S | ISO-8601 | |

#### Default quotas by plan (lab)

| Plan | `dailyQuota` | Notes |
|---|---|---|
| Free | `3` | Nearby / short trips only (enforced in Lambda) |
| Plus | `10` | Far + multi-day allowed |
| Pro | `999` or high number | Unlimited for lab practicality |

#### Step 1 — Create the table

1. Open **DynamoDB** → **Tables** → **Create table**
2. Configure:
   - **Table name:** `TripAITravelPlan`
   - **Partition key:** `userId` · type **String**
   - **Table settings:** Customize settings (or defaults)
   - **Capacity mode:** **On-demand**
   - Encryption: AWS owned key (default)
3. Click **Create table**

<!-- TODO: screenshot - form Create table wakan-users -->
![Create wakan-users table](/worlog/images/5-Workshop/5.5-Data-layer/users-create.png)

4. Wait until status is **Active**

<!-- TODO: screenshot - table Active trong danh sách -->
![wakan-users Active](/worlog/images/5-Workshop/5.5-Data-layer/users-active1.png)

#### Step 2 — Insert a sample user (test)

1. Open table `TripAITravelPlan` → **Explore table items** → **Create item**
2. Use JSON view and paste (replace `userId` with your Cognito test user’s **sub**):

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

3. Click **Create item**

<!-- TODO: screenshot - item Free user trong table -->
![Sample Free user item](/worlog/images/5-Workshop/5.5-Data-layer/users-sample-item.png)

**How to get Cognito `sub`:**

- After Hosted UI login, decode the JWT (jwt.io) → claim `sub`, **or**
- Cognito → Users → open the user → **User sub** / attributes

#### Step 3 — (Optional) second sample for Plus / Pro

Create another item with the same structure but:

```json
"plan": { "S": "pro" },
"dailyQuota": { "N": "999" },
"remainingQuota": { "N": "999" }
```

Useful later when testing Pro + Verified Places.

#### CLI alternative

```bash
aws dynamodb create-table \
  --table-name wakan-users \
  --attribute-definitions AttributeName=userId,AttributeType=S \
  --key-schema AttributeName=userId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-southeast-1
```

#### Checkpoint

| Item | Expected result |
|---|---|
| Table | `wakan-users` status **Active** |
| PK | `userId` (String) |
| Capacity | On-demand |
| Sample item | At least one Free user with `remainingQuota` |
| Cognito link | `userId` = real Cognito `sub` of test user |
