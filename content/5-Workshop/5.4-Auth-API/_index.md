---
title: "Authentication & API Layer"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Goal

In this section, you will build the authentication and API entry layer of Wakan:

- **Amazon Cognito** for sign-up, sign-in, and JWT tokens
- **Amazon API Gateway** as the REST API front door
- **Cognito Authorizer** so only signed-in users can call protected routes
- **Usage Plans** for technical rate limiting (anti-spam)
- **CloudFront `/api/*` behavior** so frontend and API share one domain

```
Browser
  │  sign-in
  ▼
Cognito Hosted UI  ──►  ID / Access token (JWT)
  │
  │  Authorization: Bearer <token>
  ▼
CloudFront  (/api/*)
  │
  ▼
API Gateway  ──► Cognito Authorizer  ──►  (Lambda in Step 5.6)
```

{{% notice note %}}
Lambda integration is completed in **Step 5.6**. In this section you create Cognito, API Gateway, the authorizer, routes, and (optionally) a **MOCK** integration to verify JWT validation works.
{{% /notice %}}

#### Content

1. [Create Cognito User Pool](5.4.1-cognito/)
2. [Create API Gateway & Cognito Authorizer](5.4.2-api-gateway/)
3. [Configure Usage Plans](5.4.3-usage-plans/)
4. [Route `/api/*` through CloudFront](5.4.4-cloudfront-api/)
