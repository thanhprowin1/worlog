---
title: "Week 8 Worklog"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:
* Build the backend scaffolding: API Gateway + Lambda AI Processor and finalize the DynamoDB schema.
* Align API contracts with the frontend and AI teams for smooth integration next week.

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Status |
| --- | --- | --- | --- | --- |
| 2 | - Sync API contract: define request/response for `/recommendation` <br> - Agree on payload format between frontend,AI Processor | 08/06/2026 | 08/06/2026 | Completed |
| 3 | - Create API Gateway REST API `wakan-api`; attach Cognito Authorizer <br> - Create `/recommendation` resource with POST method pointing to Lambda AI Processor | 09/06/2026 | 09/06/2026 | Completed |
| 4 | - Finalize DynamoDB schema: Users/Subscriptions (PK userId, attrs tier/usedToday/usedThisMonth/resetAt) <br> - RecommendationCache (PK cacheKey, TTL expiresAt) | 10/06/2026 | 10/06/2026 | Completed |
| 5 | - Write skeleton Lambda AI Processor: parse API Gateway event, extract userId + tier from Cognito claims <br> - Deploy to AWS and test via Console Test event | 11/06/2026 | 11/06/2026 | Completed |
| 6 | - Attach IAM policy for Lambda: dynamodb:GetItem/PutItem/UpdateItem + lambda:InvokeFunction <br> - Log parse and claim-extraction steps to CloudWatch | 12/06/2026 | 12/06/2026 | Completed |

### Week 8 Achievements:
* **API Gateway + Orchestrator skeleton:** The `/recommendation` endpoint receives requests, parses userId/tier from the Cognito token, and returns a sample response — no quota/cache logic yet, but the scaffold runs.
* **Complete DynamoDB schema:** Both Users/Subscriptions and RecommendationCache tables are fully designed and ready for business logic implementation next week.