---
title: "Technical Blogs"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section lists and introduces the technical blogs I have researched, composed, and translated during my learning journey. These articles reflect my hands-on experience in tackling real-world cloud computing challenges.

### [Blog 1 - Don't let EC2 run overnight: Drastically reduce AWS costs with Spot Instances, Auto Scaling, and Scheduler](3.1-Blog1/)
This blog explores strategies to optimize cloud costs for development and testing environments. It details how to leverage AWS Spot Instances for up to 90% savings, handle interruptions gracefully by decoupling state, ensure high availability using Auto Scaling Groups with Capacity Rebalancing, and automate resource lifecycles with AWS Instance Scheduler.

---

### [Blog 2 - Preventing "forgot to turn off" AWS resources: Budgets, Cost Anomaly Detection, and Tagging](3.2-Blog2/)
This blog focuses on establishing essential Cost Governance for AWS beginners. It provides a step-by-step guide to setting up automated financial guardrails, including AWS Budgets for forecasted limits, machine learning-powered Cost Anomaly Detection for sudden spikes, and strict Tagging rules for precise cost allocation and resource management.

---

### [Blog 3 - Designing Stateless applications on AWS: Why you shouldn't store files directly on EC2](3.3-Blog3/)
This blog delves into cloud-native architectural best practices, specifically the importance of building stateless applications. You will learn how to decouple your architecture by migrating static files to Amazon S3, structured data to Amazon RDS, and system logs to Amazon CloudWatch, enabling seamless horizontal scaling and risk-free Spot Instance adoption.