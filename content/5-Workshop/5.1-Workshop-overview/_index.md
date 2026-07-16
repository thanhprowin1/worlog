---
title: "Workshop Overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### What you will build

In this workshop, you will deploy the core serverless architecture of **Wakan** — an AI-powered travel itinerary generator for Vietnam. By the end, you will have a working end-to-end flow:

> User opens website → signs in via Cognito → selects trip preferences on UI → API Gateway routes to Lambda Orchestrator → Orchestrator checks cache & quota → calls Lambda AI Processor → returns structured itinerary.

#### Architecture at a glance

<!-- TODO: screenshot/diagram - kiến trúc tổng quan Wakan -->
![Wakan architecture](/worlog/images/5-Workshop/5.1-Workshop-overview/architecture.jpg)

The system is organized into 4 layers:

**Layer 1 — Edge & Frontend**
- Amazon S3 hosts the static web application (HTML/CSS/JS).
- Amazon CloudFront distributes content globally and forwards `/api/*` requests to API Gateway.
- AWS WAF (optional) filters malicious HTTP/HTTPS traffic at the edge.

**Layer 2 — Authentication & API**
- Amazon Cognito manages user sign-up, sign-in, and JWT tokens.
- API Gateway Cognito Authorizer validates every incoming request before forwarding it.
- Usage Plans enforce technical rate limits per subscription tier.

**Layer 3 — Business Logic (Lambda)**
- **Orchestrator Lambda:** validates input, checks user quota from DynamoDB, generates cache key from rounded coordinates + preferences, serves cached result if available, otherwise delegates to AI Processor.
- **AI Processor Lambda:** retrieves the AI API key from Secrets Manager, builds a structured prompt, calls the external AI model, enforces JSON schema output, and handles fallback errors.

**Layer 4 — Data (DynamoDB)**
- **Users/Subscriptions:** user info, current plan (Free/Plus/Pro), remaining daily quota.
- **Cache:** pre-generated itineraries keyed by rounded location + preferences; TTL varies by trip length (6–12h for short trips, 24–48h for multi-day).
- **Verified Places:** curated location data used exclusively by the Pro plan to prevent AI hallucination.

#### How requests flow

<!-- TODO: screenshot/diagram - sequence diagram flow request -->


1. User visits `www.wakan.id.vn` — CloudFront serves static assets from S3.
2. User signs in through Cognito Hosted UI → receives ID token + access token.
3. User selects trip options on the UI (companions, budget, days, transport, experience type).
4. Frontend sends POST `/api/itinerary` with token in Authorization header.
5. API Gateway validates JWT via Cognito Authorizer → forwards to Orchestrator Lambda.
6. Orchestrator reads user plan & quota from DynamoDB → generates cache key.
7. If cache hit → return itinerary immediately (no AI call needed).
8. If cache miss → invoke AI Processor Lambda.
9. AI Processor fetches API key from Secrets Manager → builds prompt → calls AI model → validates JSON output → saves result to Cache table → returns to Orchestrator.
10. Orchestrator decrements quota in DynamoDB → returns itinerary to frontend.


