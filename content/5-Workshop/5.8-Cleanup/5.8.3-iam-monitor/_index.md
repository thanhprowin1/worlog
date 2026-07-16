---
title: "Delete IAM, monitoring & final check"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.8.3. </b> "
---

#### Objective

Remove leftover IAM roles, CloudWatch artifacts, optional Budgets, and run a final cost/resource check.

#### Step 1 — IAM roles (Lambda execution roles)

1. **IAM** → **Roles**
2. Delete lab roles (detach policies first if required):
   - `wakan-ai-processor-role` (or your name)
3. If you created a custom workshop policy, delete it from **Policies** if unused

<!-- TODO: screenshot - IAM roles cleaned -->
![Delete IAM roles](/worlog/images/5-Workshop/5.8-Cleanup/iam-delete.png)

{{% notice note %}}
You cannot delete a role still attached to a Lambda. Delete functions (Step 5.8.2) first.
{{% /notice %}}

#### Step 2 — CloudWatch

| Artifact | Action |
|---|---|
| Alarms (`wakan-orchestrator-errors`, …) | Delete alarm |
| Dashboard `Wakan-Overview` | Delete dashboard |
| SNS topic `wakan-alerts` | Delete subscriptions → delete topic |
| Log groups `/aws/lambda/wakan-*` | Delete log groups (optional but stops storage cost) |

1. CloudWatch → **Alarms** → select → **Delete**
2. **Dashboards** → delete `Wakan-Overview`
3. **SNS** → delete topic `wakan-alerts` (if created)
4. **Log groups** → delete  `/aws/lambda/TripAIBackend`

<!-- TODO: screenshot - alarms/dashboard/log groups cleaned -->
![Clean CloudWatch](/worlog/images/5-Workshop/5.8-Cleanup/cw-clean.png)

```bash
aws logs delete-log-group --log-group-name /aws/lambda/wakan-orchestrator --region ap-southeast-1
aws logs delete-log-group --log-group-name /aws/lambda/wakan-ai-processor --region ap-southeast-1
```

#### Step 3 — AWS Budgets (optional)

1. **Billing** → **Budgets**
2. Delete `wakan-monthly-budget` if you no longer need alerts
3. Or keep it if you still run other workloads on the account

<!-- TODO: screenshot - budget deleted or kept with note -->


#### Step 4 — Final verification checklist

Search the console (or CLI) for leftover `wakan` resources:

| Service | Check |
|---|---|
| CloudFront | No `wakan` distributions |
| S3 | No `wakan-frontend-*` bucket |
| API Gateway | No `wakan-api` |
| Lambda | No `wakan-*` functions |
| Cognito | No `wakan-users` pool |
| DynamoDB | No `wakan-*` tables |
| Secrets Manager | No `wakan/ai-api-key` |
| WAF | No `wakan-web-acl` |
| IAM | No lab execution roles left |
| CloudWatch | No lab alarms / log groups (if deleted) |

```bash
# Quick CLI scans (examples)
aws lambda list-functions --region ap-southeast-1 --query "Functions[?contains(FunctionName, 'wakan')].FunctionName"
aws dynamodb list-tables --region ap-southeast-1 --query "TableNames[?contains(@, 'wakan')]"
aws s3 ls | findstr wakan
```

<!-- TODO: screenshot - CLI or console search shows no wakan leftovers -->


#### Step 5 — Cost Explorer (next day)

1. After 24 hours, open **Cost Explorer**
2. Confirm Wakan-related spend is flat / only residual (CloudFront occasionally has short delay)
3. Keep monitoring external AI provider billing separately

#### Workshop complete

You have finished the Wakan hands-on path:

| Step | Topic |
|---|---|
| 5.1 | Overview & architecture |
| 5.2 | Prerequisites |
| 5.3 | Frontend & edge (S3, CloudFront, WAF) |
| 5.4 | Auth & API (Cognito, API Gateway, Usage Plans) |
| 5.5 | Data layer (DynamoDB) |
| 5.6 | Business logic (Orchestrator + AI Processor) |
| 5.7 | Security, monitoring & budgets |
| 5.8 | Cleanup |

{{% notice tip %}}
For your internship report, keep the **screenshots taken before cleanup**. After deletion, you cannot re-capture production console evidence easily without redeploying.
{{% /notice %}}
