---
title: "Security, Monitoring & Cost Control"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Goal

In this section you will harden the security posture of Wakan, set up observability with CloudWatch, and configure cost guardrails so AI spend does not spiral out of control.

```
Security                          Monitoring                    Cost Control
─────────                         ───────────                   ──────────────
• IAM least privilege             • CloudWatch Logs             • AWS Budgets alerts
• Secrets Manager                 • Custom metrics              • Cache → reduce AI calls
• Rounded coordinates (privacy)   • API Gateway metrics         • Quota per tier
• WAF at edge                     • Lambda metrics              • IAM policy deny on delete
```

{{% notice note %}}
This step is **observability and hardening** — no new resources are created that change the functional flow. You are adding guardrails around what you built in Steps 5.1–5.6.
{{% /notice %}}

#### Content

1. [IAM least privilege review](5.7.1-iam-review/)
2. [CloudWatch monitoring](5.7.2-cloudwatch/)
3. [AWS Budgets alert](5.7.3-budgets/)
