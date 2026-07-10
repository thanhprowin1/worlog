---
title: "AWS Budgets alert"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.7.3. </b> "
---

#### Objective

Create an **AWS Budget** with email alerts so you know when monthly spend approaches a limit. AI API calls (external provider) plus AWS usage (Lambda, API Gateway, CloudFront, DynamoDB) can grow quickly under load.

{{% notice note %}}
AWS Budgets tracks **AWS service costs**. External AI provider bills (OpenAI, Anthropic, …) are **outside** AWS — monitor those in the provider dashboard separately. Cache + quota in Wakan are the main levers for AI cost.
{{% /notice %}}

#### Cost levers already in the architecture

| Lever | Where | Effect |
|---|---|---|
| Cache HIT | DynamoDB + Orchestrator | Avoids AI call entirely |
| Usage Plan throttle | API Gateway | Caps request rate |
| WAF | CloudFront | Blocks bot spam before it hits Lambda |
| Timeout / fallback | AI Processor | Bounds hanging expensive calls |

#### Step 1 — Create a monthly cost budget

1. Open **AWS Billing** → **Budgets** → **Create budget**
   *(or search “Budgets” in the console)*
2. Choose **Use a template** or **Customize**
3. Recommended lab settings:

| Setting | Value |
|---|---|
| **Budget type** | Cost budget |
| **Name** | `wakan-monthly-budget` |
| **Period** | Monthly |
| **Budget amount** | e.g. `$10` (lab) or `$50` (demo) |
| **Budget scope** | All AWS services (or filter by tag if you tagged resources) |

<!-- TODO: screenshot - Create budget form -->
![Create budget](/images/5-Workshop/5.7-Security-monitoring/budget-create.png)

#### Step 2 — Configure alerts

Add thresholds:

| Threshold | Trigger | Action |
|---|---|---|
| **50%** of budget | Actual cost | Email warning |
| **80%** of budget | Actual cost | Email urgent |
| **100%** of budget | Actual cost | Email critical |
| **100%** of budget | Forecasted cost | Email early warning |

1. Enter your email for each alert
2. Create budget

<!-- TODO: screenshot - alert thresholds 50/80/100 -->
![Budget alerts](/images/5-Workshop/5.7-Security-monitoring/budget-alerts.png)

#### Step 3 — (Optional) Tag resources for filtered budgets

If you want a budget **only for Wakan**:

1. Tag all Wakan resources with:
   - Key: `Project`
   - Value: `Wakan`
2. Create a budget filtered by that tag

```bash
# Example: tag a Lambda function
aws lambda tag-resource \
  --resource arn:aws:lambda:ap-southeast-1:<ACCOUNT_ID>:function:wakan-orchestrator \
  --tags Project=Wakan \
  --region ap-southeast-1
```

Tag S3, CloudFront, DynamoDB tables, API Gateway, Secrets, WAF the same way for accurate cost attribution.

<!-- TODO: screenshot - resource tags Project=Wakan -->

#### Step 4 — Confirm SNS / email (if required)

Some budget notifications use SNS. Confirm any subscription email so alerts are not stuck in “Pending confirmation”.

#### Step 5 — Document external AI monitoring

Add a short note in your report:

| Provider | Where to monitor | Suggested action |
|---|---|---|
| OpenAI / Anthropic / other | Provider billing dashboard | Set hard monthly cap if available |
| AWS | Budgets + Cost Explorer | `wakan-monthly-budget` |

#### Checkpoint

| Item | Expected result |
|---|---|
| Budget exists | Name `wakan-monthly-budget`, monthly period |
| Alerts | At least 50% and 100% email alerts |
| Email confirmed | You receive a confirmation / test if available |
| Tags (optional) | `Project=Wakan` on core resources |
| External AI | Documented separately from AWS Budgets |

#### Section 5.7 summary

| Area | What you did |
|---|---|
| **Security** | Reviewed least-privilege IAM; optional deny-delete; secret rotation note |
| **Monitoring** | Log retention, error alarms, optional cache metrics, dashboard |
| **Cost** | Monthly budget + multi-threshold email alerts |

Next: **Step 5.8 Cleanup** — delete resources in a safe order to stop all charges.
