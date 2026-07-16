---
title: "Blog 2: Preventing 'forgot to turn off' AWS resources"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Preventing "forgot to turn off" AWS resources: Budgets, Cost Anomaly Detection, and Tagging

When getting started with AWS, the biggest fear for many isn't code bugs, but rather being charged unexpectedly for forgetting to turn off resources. 

A familiar scenario: you create an EC2 instance for a lab, spin up RDS to test a database, and then accidentally leave them running in the background for a week. By the end of the month, your billing dashboard reveals an unexpected surge in costs. To prevent this, you should build a basic Cost Governance layer from day one using three tools: **AWS Budgets, AWS Cost Anomaly Detection, and Cost Allocation Tags.**

---

## Solution Highlights

### 1. Build an Early Warning System with AWS Budgets
* **The Problem:** Checking bills manually every day is impractical and easily forgotten.
* **The Solution:** Use **AWS Budgets** to set a fixed monthly budget (e.g., a $10/month limit for a lab account).
* **Optimization:** This service doesn't just alert you based on Actual costs, but also Forecasted costs. If you've only spent $2, but the algorithm forecasts you'll exceed the $10 threshold by month's end, you will receive an early warning email.

![AWS Budgets Dashboard](/worlog/images/Blog2.jpg)

### 2. Detect Irregularities with AWS Cost Anomaly Detection
* **Core Risk:** With a new account, forecasted alerts might not be highly accurate initially. If your Access Key is leaked or a script provisions expensive resources by mistake, costs will spike rapidly before a Budget Alert triggers.
* **The Solution:** Activate **AWS Cost Anomaly Detection**. This service uses advanced Machine Learning models to continuously monitor and learn your account's daily spending patterns.
* **The Result:** The system sends early alerts when it detects anomalous spending, protecting you from resource abuse and costly mistakes.

### 3. Categorize and Isolate Costs with Tagging
* **The Problem:** When running multiple projects concurrently, a consolidated bill makes it difficult to distinguish which service belongs to which project.
* **The Solution:** Establish strict **Tagging** rules from the moment resources are provisioned (e.g., `Project = GraduationThesis`, `Environment = Dev`, `Purpose = Lab`).
* **Technical Note:** Tags must be activated in the *Cost Allocation Tags* section of the Billing console to be used for cost allocation.

---

## Conclusion
Cloud Cost Management (FinOps) shouldn't be about waiting until the end of the month to check your bill and feel regret. A financially secure AWS account requires not just knowing how to deploy resources, but knowing how to monitor, categorize, and clean them up promptly.