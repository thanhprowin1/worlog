---
title: "Week 4 Worklog"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:
* Move from Console clicks to automation: Lambda, Systems Manager, and IaC (CloudFormation & CDK).
* Combine advanced monitoring with FinOps so operations stay cost-sustainable.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Write Lambda handlers for S3 events / EventBridge schedules <br> - Access EC2 securely via Systems Manager Session Manager (no public SSH) | 11/05/2026 | 11/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Declare a CloudFormation stack (minimal VPC + EC2) <br> - Compare the YAML template with manual Week 1 steps | 12/05/2026 | 12/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Rebuild the same stack with the AWS CDK (TypeScript/Python) <br> - Install the AWS Toolkit for VS Code and deploy from the IDE | 13/05/2026 | 13/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Build advanced CloudWatch dashboards; try a Grafana data source <br> - Enable VPC Flow Logs; schedule snapshots with EBS Data Lifecycle Manager | 14/05/2026 | 14/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Analyze Savings Plans and Reserved Instances for the lab <br> - Query cost data with Glue Crawlers + Athena | 15/05/2026 | 15/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 4 Achievements:
* **Automated operations:** Lambda replaces repetitive work; SSM manages hosts without an internet-facing bastion.
* **Reproducible infrastructure:** The same stack can be rebuilt with CloudFormation or CDK and kept in source control.
* **Observability + cost:** Flow Logs / CloudWatch for visibility; Savings Plans/RI and Athena for structured bill analysis.
