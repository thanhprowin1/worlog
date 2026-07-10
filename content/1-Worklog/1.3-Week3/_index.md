---
title: "Week 3 Worklog"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:
* Build a defensive ring around apps: user identity, edge firewall, encryption, and threat detection.
* Connect multi-VPC networks and see how Lightsail simplifies compute before a full VPC stack is needed.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Create a Cognito User Pool for sign-up / sign-in <br> - Compare Cognito with enterprise SSO flows | 04/05/2026 | 04/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Attach AWS WAF to CloudFront; try sample XSS/SQLi rules <br> - Overview of AWS Firewall Manager for centralized control | 05/05/2026 | 05/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Create and use a customer managed key with AWS KMS <br> - Scan for sensitive data with Amazon Macie (lab) | 06/05/2026 | 06/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Enable Amazon GuardDuty and review sample findings <br> - Connect multi-VPC networks with Transit Gateway | 07/05/2026 | 07/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Deploy a quick demo app on Amazon Lightsail <br> - Contrast Lightsail with a self-built EC2 + VPC setup | 08/05/2026 | 08/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 3 Achievements:
* **Layered defense:** Identity (Cognito/SSO), edge (WAF), data (KMS/Macie), and detection (GuardDuty) form one defensive chain.
* **Network scale-out:** Transit Gateway clarifies topology when multiple VPCs / environments exist.
* **Compute choice:** Know when Lightsail is enough and when a full EC2 + VPC design is the better fit.
