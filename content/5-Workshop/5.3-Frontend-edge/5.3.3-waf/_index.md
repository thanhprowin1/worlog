---
title: "Attach WAF (optional)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

#### Objective

Attach **AWS WAF** to the CloudFront distribution to filter common web attacks (SQL injection, XSS, bad bots) before traffic reaches your origin.

{{% notice note %}}
This step is **optional** for the MVP. You can skip it and continue to Step 5.4 if you want to finish the core flow first. WAF adds a small cost (Web ACL + request fees).
{{% /notice %}}

#### Step 1 — Create a Web ACL

1. Open **AWS WAF & Shield** → **Web ACLs** → **Create web ACL**
2. Configure:
   - **Resource type:** Amazon CloudFront distributions  
     *(WAF for CloudFront is managed in **us-east-1** even if your other resources are in Singapore)*
   - **Name:** `TripAI-Web-Firewall`
   - **CloudWatch metric name:** `wakanWebAcl`

<!-- TODO: screenshot - form Create web ACL (resource type CloudFront) -->
![Create Web ACL](/images/5-Workshop/5.3-Frontend-edge/waf-create.png)

3. Click **Next**

#### Step 2 — Add managed rule groups

On the **Add rules and rule groups** page, click **Add rules** → **Add managed rule groups**.

Enable these AWS Managed Rules (recommended for a public web app):

| Rule group | Purpose |
|---|---|
| **AWSManagedRulesCommonRuleSet** | Core protections (XSS, bad inputs, …) |
| **AWSManagedRulesKnownBadInputsRuleSet** | Known malicious patterns |
| **AWSManagedRulesAmazonIpReputationList** | Blocks IPs with bad reputation / bots |

For each rule group:
- Set **Action** to the default managed action (usually Block where applicable)
- Keep capacity within the Web ACL limit (1,500 WCU by default)

<!-- TODO: screenshot - chọn managed rule groups -->
![Managed rules](/images/5-Workshop/5.3-Frontend-edge/waf-rules.png)

Click **Add rules** → **Next**.

#### Step 3 — Default action & review

1. **Default web ACL action:** Allow  
   *(only requests matching a Block rule are denied)*
2. Review settings → **Create web ACL**

<!-- TODO: screenshot - Web ACL created successfully -->
![Web ACL created](/images/5-Workshop/5.3-Frontend-edge/waf-created.png)

#### Step 4 — Associate with CloudFront

1. Open the Web ACL you just created → **Associated AWS resources** → **Add AWS resources**
2. Select your CloudFront distribution
3. Click **Add**

Alternatively, from **CloudFront** → your distribution → **Security** → **Edit** → choose the Web ACL `TripAI-Web-Firewall` → Save.

<!-- TODO: screenshot - Web ACL đã associate với CloudFront distribution -->
![WAF associated](/images/5-Workshop/5.3-Frontend-edge/waf-associated.png)

#### Step 5 — Quick verification

1. Open the CloudFront URL in a browser — the site should still load normally
2. (Optional) Check **WAF → Web ACL → Overview** after a few requests to see Sampled requests / metrics

#### Checkpoint

| Item | Expected result |
|---|---|
| Web ACL exists | Name `TripAI-Web-Firewall`, resource type CloudFront |
| Managed rules | At least Common + KnownBadInputs + IP Reputation |
| Association | Linked to your CloudFront distribution |
| Site still works | Frontend loads over HTTPS as before |

{{% notice tip %}}
**Cost tip:** If you only need WAF for a demo/presentation, create it, take screenshots, then remove the association and delete the Web ACL in the Cleanup step to avoid ongoing request charges.
{{% /notice %}}
