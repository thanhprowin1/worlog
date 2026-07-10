---
title: "Clean up"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Goal

Delete all Wakan workshop resources in a **safe order** so you stop ongoing charges and avoid dependency errors (e.g. deleting a bucket that CloudFront still uses).

{{% notice warning %}}
Cleanup is **irreversible**. Export any screenshots, logs, or sample data you need for your report **before** deleting resources.
{{% /notice %}}

#### Recommended delete order

```
1. CloudFront  (disable → delete distribution; remove WAF association)
2. WAF         (Web ACL)
3. API Gateway (API + usage plans + API keys)
4. Lambda      ( AI Processor)
5. Cognito     (User pool + domain)
6. DynamoDB    (tables)
7. Secrets Manager
8. S3          (empty bucket → delete)
9. IAM roles & CloudWatch artifacts (alarms, dashboard, log groups)
10. AWS Budgets (optional)
```

#### Content

1. [Delete edge & API resources](5.8.1-edge-api/)
2. [Delete compute, data & secrets](5.8.2-compute-data/)
3. [Delete IAM, monitoring & final check](5.8.3-iam-monitor/)
