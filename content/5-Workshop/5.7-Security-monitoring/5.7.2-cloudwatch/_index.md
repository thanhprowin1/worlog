---
title: "CloudWatch monitoring"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

#### Objective

Set up CloudWatch Logs retention, key metrics, and a simple dashboard so you can observe:

- How many API requests hit the system
- Cache hit vs cache miss ratio
- Lambda errors and duration
- AI call failures

#### Metrics that matter for Wakan

| Metric | Source | Why it matters |
|---|---|---|
| `API 4XXError` / `5XXError` | API Gateway | Auth failures / server errors |
| `Count` (requests) | API Gateway | Traffic volume |
| `Invocations` | Lambda Orchestrator | Total request volume |
| `Errors` | Lambda Orchestrator / AI Processor | Runtime failures |
| `Duration` | Lambda AI Processor | AI latency (cost + UX) |
| `Throttles` | Lambda / API Gateway | Capacity pressure |
| Cache HIT / MISS | Custom metric (optional) | AI cost control effectiveness |
| `remainingQuota` hits 0 | Custom or DynamoDB stream (advanced) | User experience |

#### Step 1 — Set log retention

By default CloudWatch Logs never expire (cost accumulates). Set retention:

1. Open **CloudWatch** → **Log groups**
2. Select `/aws/lambda/wakan-orchestrator` → **Actions** → **Edit retention setting**
3. Set retention to **7 days** (lab) or **30 days** (demo)
4. Repeat for `/aws/lambda/wakan-ai-processor`

<!-- TODO: screenshot - log retention set to 7 days -->
![Log retention](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-retention.png)

CLI alternative:

```bash
aws logs put-retention-policy \
  --log-group-name /aws/lambda/wakan-orchestrator \
  --retention-in-days 7 \
  --region ap-southeast-1

aws logs put-retention-policy \
  --log-group-name /aws/lambda/wakan-ai-processor \
  --retention-in-days 7 \
  --region ap-southeast-1
```

#### Step 2 — Create a CloudWatch Alarm for Lambda errors

1. CloudWatch → **Alarms** → **Create alarm**
2. Select metric:
   - Namespace: `AWS/Lambda`
   - Metric name: `Errors`
   - FunctionName: `wakan-orchestrator`
3. Configure:
   - Statistic: Sum
   - Period: 5 minutes
   - Threshold: ≥ 1 (or 5 for less noise)
4. Notification:
   - Create SNS topic `wakan-alerts`
   - Subscribe your email
5. Alarm name: `wakan-orchestrator-errors`

<!-- TODO: screenshot - alarm create form -->
![Create alarm](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-alarm.png)

6. Confirm the SNS subscription email

Repeat optionally for `wakan-ai-processor` Errors and for API Gateway `5XXError`.

#### Step 3 — Emit custom cache metrics (optional but recommended)

In the Orchestrator code, after cache HIT / MISS, add:

```python
import boto3
cloudwatch = boto3.client("cloudwatch")

def put_cache_metric(hit: bool):
    cloudwatch.put_metric_data(
        Namespace="Wakan",
        MetricData=[
            {
                "MetricName": "CacheHit" if hit else "CacheMiss",
                "Value": 1,
                "Unit": "Count",
            }
        ],
    )
```

Call `put_cache_metric(True)` on HIT and `put_cache_metric(False)` on MISS.

Grant the Orchestrator role:

```json
{
  "Effect": "Allow",
  "Action": ["cloudwatch:PutMetricData"],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "cloudwatch:namespace": "Wakan"
    }
  }
}
```

<!-- TODO: screenshot - custom Wakan namespace metrics after a few requests -->


#### Step 4 — Create a simple dashboard

1. CloudWatch → **Dashboards** → **Create dashboard** → name `Giai_doan_1`
2. Add widgets:

| Widget | Type | Content |
|---|---|---|
| API requests | Line | API Gateway `Count` for `wakan-api` / stage `prod` |
| Lambda errors | Number | Orchestrator + AI Processor `Errors` |
| AI duration | Line | AI Processor `Duration` p50 / p99 |
| Cache HIT vs MISS | Stacked | Custom `Wakan/CacheHit` and `Wakan/CacheMiss` |

3. Save dashboard

<!-- TODO: screenshot - Wakan-Overview dashboard -->
![CloudWatch dashboard](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-dashboard.png)

#### Step 5 — Sample request log inspection

1. Generate a few test requests (Step 5.6.3)
2. Open log group → latest stream
3. Verify structured logs look like:

```json
{"level":"info","msg":"cache_hit","cacheKey":"a1b2c3...","userId":"sub-xxx"}
{"level":"error","msg":"AI call failed","error":"timeout"}
```

{{% notice important %}}
Never log: API keys, full Authorization headers, precise GPS coordinates, or raw AI prompts that might contain sensitive data. Log only rounded coordinates and option enums.
{{% /notice %}}

<!-- TODO: screenshot - clean structured log (no secrets) -->
![Clean logs](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-logs-clean.png)

#### Checkpoint

| Item | Expected result |
|---|---|
| Log retention | 7 or 30 days on both Lambda log groups |
| Alarm | At least Orchestrator Errors → SNS email |
| Custom metrics (optional) | `Wakan/CacheHit` and `Wakan/CacheMiss` appear |
| Dashboard | `Wakan-Overview` shows traffic + errors |
| Log hygiene | No secrets or precise GPS in logs |
