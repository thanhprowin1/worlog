---
title: "IAM least privilege review"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---

#### Objective

Review and tighten IAM roles so each component has only the permissions it needs. This is a **security best practice** — not a new resource to create.

#### Role summary table

| Role | Attached to | Permissions | Denied |
|---|---|---|---|
| `wakan-orchestrator-role` | `wakan-orchestrator` Lambda | DynamoDB read/write on `wakan-users` + `wakan-cache`; Lambda invoke on `wakan-ai-processor` | None needed |
| `wakan-ai-processor-role` | `wakan-ai-processor` Lambda | Secrets Manager `GetSecretValue` on `wakan/ai-api-key`; DynamoDB Query on `wakan-verified-places` | No write access to users/cache tables |
| `workshop-admin-role` | Your IAM user/role | Policy from Step 5.2 (create + cleanup all resources) | N/A |

#### Step 1 — Verify Orchestrator role

1. Open **IAM** → **Roles** → search `wakan-orchestrator-role` (or whatever name you used)
2. Check **Permissions policy**:

```jsonc
{
  // Should contain ONLY these actions:
  // - logs:CreateLogGroup, logs:CreateLogStream, logs:PutLogEvents
  // - dynamodb:GetItem, dynamodb:UpdateItem, dynamodb:Query   → wakan-users ARN
  // - dynamodb:GetItem, dynamodb:PutItem                       → wakan-cache ARN
  // - lambda:InvokeFunction                                    → wakan-ai-processor ARN
}
```

<!-- TODO: screenshot - Orchestrator IAM policy -->
![Orchestrator IAM](/worlog/images/5-Workshop/5.7-Security-monitoring/iam-orchestrator.png)

#### Step 2 — Verify AI Processor role

1. Open **IAM** → **Roles** → search `wakan-ai-processor-role`
2. Check **Permissions policy**:

```jsonc
{
  // Should contain ONLY these actions:
  // - logs:*Log*
  // - secretsmanager:GetSecretValue    → wakan/ai-api-key ARN
  // - dynamodb:Query, dynamodb:GetItem → wakan-verified-places ARN
}
```

Confirm it does **not** have any of these:
- ❌ `dynamodb:PutItem` on `wakan-users` or `wakan-cache`
- ❌ `secretsmanager:CreateSecret`, `DeleteSecret` (only `GetSecretValue`)

<!-- TODO: screenshot - AI Processor IAM policy -->


#### Step 3 — Add SCP deny on deletion (optional but recommended)

To prevent accidental deletion during the workshop period, add an **Organizational SCP** or an **IAM policy** that denies destructive actions unless authorized by a specific group (e.g., `workshop-admins`).

Example inline policy for non-admin users:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PreventDestructiveActions",
      "Effect": "Deny",
      "Action": [
        "dynamodb:DeleteTable",
        "lambda:DeleteFunction",
        "apigateway:DELETE",
        "cloudfront:DeleteDistribution"
      ],
      "Resource": "*"
    }
  ]
}
```

This forces deliberate action before deleting resources — useful during presentations or team workshops.

<!-- TODO: screenshot - SCP / deny policy attached -->



For production, rotate the AI API key regularly and store it as a versioned secret:

```bash
# Example: rotate by creating a new version label
aws secretsmanager put-secret-value \
  --secret-id wakan/ai-api-key \
  --secret-string '{"api_key":"NEW_KEY_HERE"}' \
  --version-stages ["AWSCURRENT"] \
  --region ap-southeast-1
```

Keep old versions for 30 days before deleting them (`AWSPREVIOUS`).

#### Checkpoint

| Item | Expected result |
|---|---|
| Orchestrator role | Can read/write DynamoDB + invoke AI Processor only |
| AI Processor role | Can read secret + query verified places only; no write to users/cache |
| Deny policy | Destructive actions blocked for non-admin users |
| Secret rotation | Documented procedure noted |
