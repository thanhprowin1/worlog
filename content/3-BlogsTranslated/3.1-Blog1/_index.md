---
title: "Blog 1: Don't let EC2 run overnight: Reducing AWS costs with Spot Instances"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Don't let EC2 run overnight: Drastically reduce AWS costs with Spot Instance + Auto Scaling + Scheduler

When deploying study projects or test environments on AWS, many people have the habit of creating an On-Demand EC2 instance. This method is simple, but if left running overnight or over the weekend, costs can escalate quickly. Maintaining On-Demand servers for test environments (Development/Staging) that do not require 24/7 uptime is a massive waste of limited budgets.

To solve this economic problem, my project skips traditional On-Demand plans for testing and instead applies **Spot Instances** combined with a flexible, fault-tolerant architecture to cut costs to the absolute minimum.

---

## Key Highlights of the Solution

### 1. Cost-Hunting Strategy with AWS Spot Instances
* **The Problem:** The fixed hourly rate for On-Demand EC2 is relatively high.
* **The Solution:** Utilize excess hardware capacity in AWS data centers via **Spot Instances**. AWS sells this capacity at highly discounted rates, sometimes up to 90% off standard On-Demand prices.
* **The Result:** For small dev/test scenarios, EC2 costs drop significantly (e.g., from tens of dollars to just a few dollars a month).

### 2. Proactive Defense: Anticipating the AWS "Reclamation"
* **Core Risk:** Because this is spare capacity, AWS can reclaim Spot capacity at any time with only a 2-minute interruption notice.
* **Solution (Stateless Design):** The key is to absolutely never store critical data directly on the local EC2 disk. All uploaded files go to Amazon S3, the database runs on Amazon RDS, and logs are centralized in CloudWatch Logs.
* **Result:** The application is entirely **Stateless**. Even if AWS reclaims the server, the system loses no core data.

![Architecture Diagram: Stateless Application with S3, RDS, CloudWatch](/worlog/images/Blog1.jpg)

### 3. Automated Infrastructure Recovery with EC2 Auto Scaling
To prevent system downtime, EC2 Spot instances are placed within an **Auto Scaling Group (ASG)**. 
Instead of complex manual scripts, I enable **Capacity Rebalancing**. The ASG proactively reacts to an `EC2 instance rebalance recommendation`, automatically finding another Availability Zone or instance type to launch a replacement server before the current one is terminated.

### 4. Smart Power Management with AWS Instance Scheduler
For dev/test environments only used during working hours, the system integrates **Instance Scheduler on AWS**. This tool automatically stops all EC2 servers, RDS, or ASGs at 7:00 PM and restarts them at 7:00 AM (Monday to Friday), putting the system into full hibernation on weekends.

---

## Conclusion
Optimizing Cloud costs isn't just about choosing a lower-tier instance and enduring a laggy app. A well-architected system requires choosing the right pricing model (Spot Market), designing for failure, storing data in the right places, and automating resource lifecycles (FinOps). 

> **Important Note:** Spot Instances are designed for Dev/Test, batch jobs, and background workers. For Production systems requiring high uptime, combine Spot with On-Demand instances, alongside ALB, Multi-AZ, and Lifecycle hooks to ensure safety.