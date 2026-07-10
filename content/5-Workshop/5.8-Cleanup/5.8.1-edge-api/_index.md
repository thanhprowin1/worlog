---
title: "Delete edge & API resources"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

#### Objective

Remove CloudFront, WAF, and API Gateway first — they sit at the edge and depend on origins behind them.

#### Step 1 — CloudFront distribution

1. Open **CloudFront** → **Distributions** → select your distribution
2. If WAF is attached: **Security** → remove Web ACL association → Save
3. **Disable** the distribution → wait until status is **Deployed** (Disabled)
4. **Delete** the distribution

<!-- TODO: screenshot - distribution Disabled then Delete -->
![Delete CloudFront](/images/5-Workshop/5.8-Cleanup/cf-delete.png)

{{% notice note %}}
You **cannot** delete a distribution while it is still Enabled. Always Disable → wait → Delete.
{{% /notice %}}

CLI sketch:

```bash
# Get distribution ID, then:
aws cloudfront get-distribution-config --id E1234567890ABC > /tmp/cf.json
# Set Enabled=false in config, send update with IfMatch ETag, wait, then:
aws cloudfront delete-distribution --id E1234567890ABC --if-match <ETAG>
```

#### Step 2 — Origin Access Control (OAC)

1. CloudFront → **Origin access** → select `wakan-oac`
2. **Delete** if no longer used by any distribution

<!-- TODO: screenshot - delete OAC -->
![Delete OAC](/images/5-Workshop/5.8-Cleanup/oac-delete.png)

#### Step 3 — WAF Web ACL (if created)

1. Open **WAF & Shield** (scope: **CloudFront** / global, often managed from **us-east-1** UI)
2. Confirm **no associated resources** remain
3. Open Web ACL `TripAI-Web-Firewall` (or your name) → **Delete**

<!-- TODO: screenshot - Web ACL deleted / empty associations -->
![Delete WAF](/images/5-Workshop/5.8-Cleanup/waf-delete.png)

#### Step 4 — API Gateway

1. **API Gateway** → **Usage plans**
   - Disassociate stages / API keys
   - Delete plans: `wakan-plan-free`, `wakan-plan-plus`, `wakan-plan-pro` (if any)
2. **API keys** → delete lab keys (`wakan-free-key`, …)
3. **APIs** → select `wakan-api` (or `TripAI-API`) → **Actions** → **Delete API**
4. Confirm deletion

<!-- TODO: screenshot - API deleted -->
![Delete API Gateway](/images/5-Workshop/5.8-Cleanup/apigw-delete.png)

#### Checkpoint

| Item | Expected |
|---|---|
| CloudFront | Distribution deleted (not only disabled) |
| OAC | Deleted or unused |
| WAF | Web ACL deleted (if used) |
| API Gateway | API + usage plans + keys gone |
