---
title: "Delete compute, data & secrets"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---

#### Objective

Delete Lambda functions, Cognito, DynamoDB tables, Secrets Manager secrets, and the S3 frontend bucket.

#### Step 1 — Lambda functions

1. Open **Lambda** → **Functions**
2. Delete in this order (optional but clear):
   1. `TripAIBackend` (callee)
3. Confirm each deletion

<!-- TODO: screenshot - Lambda functions list empty of wakan-* -->
![Delete Lambdas](/images/5-Workshop/5.8-Cleanup/lambda-delete.png)

```bash
aws lambda delete-function --function-name wakan-orchestrator --region ap-southeast-1
aws lambda delete-function --function-name wakan-ai-processor --region ap-southeast-1
```

#### Step 2 — Cognito User Pool

1. **Amazon Cognito** → **User pools** → `User pool - weu7cv` (or your pool name)
2. **App integration** → **Domain** → delete Cognito domain first (if present)
3. **Delete user pool** → type confirm name

<!-- TODO: screenshot - user pool deleted -->
![Delete Cognito](/images/5-Workshop/5.8-Cleanup/cognito-delete.png)

{{% notice note %}}
You may need to delete the **domain** before the pool can be removed. App clients are deleted with the pool.
{{% /notice %}}

#### Step 3 — DynamoDB tables

Delete all three lab tables:

| Table |
|---|
| `wakan-users` |
| `wakan-cache` |
| `wakan-verified-places` |

1. DynamoDB → **Tables** → select table → **Delete**
2. Type the table name to confirm
3. Repeat for each table

<!-- TODO: screenshot - tables deleted / list without wakan-* -->

```bash
aws dynamodb delete-table --table-name wakan-users --region ap-southeast-2
aws dynamodb delete-table --table-name wakan-cache --region ap-southeast-2
aws dynamodb delete-table --table-name wakan-verified-places --region ap-southeast-2
```

#### Step 4 — Secrets Manager

1. **Secrets Manager** → secret `wakan/ai-api-key`
2. **Actions** → **Delete secret**
3. For lab: choose **Force delete without recovery** (or set short recovery window 7 days)

<!-- TODO: screenshot - secret scheduled for deletion / deleted -->
![Delete secret](/images/5-Workshop/5.8-Cleanup/secrets-delete.png)

```bash
aws secretsmanager delete-secret \
  --secret-id wakan/ai-api-key \
  --force-delete-without-recovery \
  --region ap-southeast-2
```

#### Step 5 — S3 bucket

S3 buckets must be **empty** before deletion.

1. Open bucket `waka.id.vn` (or your name)
2. Select all objects → **Delete** (empty the bucket completely, including versions if versioning was on)
3. Bucket → **Delete** → type bucket name

<!-- TODO: screenshot - empty bucket then delete -->
![Delete S3](/images/5-Workshop/5.8-Cleanup/s3-delete.png)

```bash
# Empty then delete (careful — irreversible)
aws s3 rm s3://waka.id.vn -<your-unique-id>/ --recursive --region ap-southeast-2
```

#### Checkpoint

| Item | Expected |
|---|---|
| Lambda | `wakan-ai-processor` gone |
| Cognito | User pool + domain deleted |
| DynamoDB | All three tables deleted |
| Secrets | `wakan/ai-api-key` deleted or pending deletion |
| S3 | Frontend bucket deleted |
