---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Wakan – AI-Powered Personalized Travel Assistant

#### Overview

**Wakan** is a serverless web application that generates personalized travel itineraries for Vietnam. Instead of free-form AI prompts, users select structured options on the UI (companions, budget, trip length, experience type, transport, nearby/far destinations). The backend validates every request, checks the user's subscription quota, serves cached results when possible, and only then calls an AI model through a dedicated processor.

This workshop walks you through building the core AWS architecture of Wakan:

- **Edge & frontend:** Amazon S3 + Amazon CloudFront (+ optional AWS WAF)
- **Auth & API:** Amazon Cognito + Amazon API Gateway
- **Business logic:**  AWS Lambda functions AI Processor
- **Data:** Amazon DynamoDB (Users/Subscriptions, Cache, Verified Places)
- **Security & cost control:** AWS Secrets Manager, least-privilege IAM, Amazon CloudWatch, AWS Budgets

By the end of this workshop, you will have a working end-to-end flow: sign in → select trip preferences → receive a structured itinerary, with quota and cache applied according to Free / Plus / Pro plans.

#### Content

1. [Workshop overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Frontend & edge distribution](5.3-Frontend-edge/)
4. [Authentication & API layer](5.4-Auth-API/)
5. [Data layer with DynamoDB](5.5-Data-layer/)
6. [Business logic with Lambda](5.6-Business-logic/)
7. [Security, monitoring & cost control](5.7-Security-monitoring/)
8. [Clean up](5.8-Cleanup/)
