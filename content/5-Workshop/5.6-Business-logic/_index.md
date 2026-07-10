---
title: "Business Logic with Lambda"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Goal

In this section, you will create and wire the two Lambda functions that form Wakan's business core:

| Function | Responsibility |
|---|---|
| **Orchestrator** | Validate input, check user plan & quota from DynamoDB, generate cache key, serve cached result if available |
| **AI Processor** | Fetch API key from Secrets Manager, build structured prompt, call external AI model, enforce JSON schema output, handle fallback |

```
Frontend POST /api/itinerary
    │
    ▼
API Gateway ──► Cognito Authorizer
    │
    ▼
Lambda Orchestrator
    ├── GetItem(wakan-users)     → plan + remainingQuota
    ├── GetItem(wakan-cache)     → cache HIT? return immediately
    │   └── MISS?
    │       │
    │       ▼
    │   Lambda AI Processor
    │       ├── GetSecretValue(Secrets Manager)
    │       ├── Call AI API (build prompt from validated options)
    │       ├── Validate JSON schema output
    │       ├── PutItem(wakan-cache)   → save for reuse
    │       └── Return itinerary
    │
    ▼
Orchestrator decrements remainingQuota in wakan-users
    │
    ▼
Response to frontend
```

{{% notice note %}}
This step replaces the **MOCK** integration from Step 5.4 with real Lambda functions. The Cognito Authorizer stays unchanged.
{{% /notice %}}

#### Content

1. [Create Lambda AI Processor](5.6.1-ai-processor/)
2. [End-to-end test](5.6.2-e2e-test/)
