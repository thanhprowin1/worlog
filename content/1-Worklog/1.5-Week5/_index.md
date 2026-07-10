---
title: "Week 5 Worklog"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:
* Package apps as containers and orchestrate them on ECS/Fargate as well as Kubernetes (EKS).
* Automate the path from code → image → cluster with CI/CD; get familiar with workflows and hybrid storage.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Build locally, and push to Amazon ECR <br> - Run a service on Amazon ECS with the Fargate launch type | 18/05/2026 | 18/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Build a CodePipeline: source → build → deploy to ECS <br> - Describe ECS cluster/service with the CDK | 19/05/2026 | 19/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Create an Amazon EKS cluster and deploy a sample workload (kubectl) <br> - Use EKS Blueprints to standardize cluster bootstrap | 20/05/2026 | 20/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Extend CI/CD for applications on EKS <br> - Overview of Red Hat OpenShift Service on AWS (ROSA) | 21/05/2026 | 21/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Orchestrate multi-step workflows with Step Functions <br> - Connect hybrid storage via Storage Gateway / Amazon FSx | 22/05/2026 | 22/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 5 Achievements:
* **Containers in practice:** Images on ECR, services on Fargate — no more hand-installed VMs as the default.
* **Two orchestration models:** Contrast simpler ECS with flexible-but-complex EKS/K8s; place ROSA in the enterprise picture.
* **Pipelines & workflows:** CodePipeline links code to clusters; Step Functions plus hybrid storage extend multi-step / on-prem integration.
