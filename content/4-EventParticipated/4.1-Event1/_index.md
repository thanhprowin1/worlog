---
title: "Event 1: Community Day"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

## Snapshot

| Item | Detail |
|------|--------|
| Name | AWS First Cloud AI Journey Community Day |
| When | 09:00 · 23 May 2026 |
| Where | 26th floor, Bitexco Tower, 02 Hai Trieu, HCMC |
| My role | Attendee |
| Speakers | Tinh Truong, Anh Pham, Thinh Nguyen, Mai Nguyen, Uyen Le, Thao Nguyen, Duc Dao, Vy Lam |

---

## What I took away

### CloudFront as more than a CDN
**Thinh Nguyen** framed CloudFront as an **edge security layer**, not only static acceleration. Notes that stuck:
- **Flat-rate pricing** reduces the risk of bill spikes during DDoS-like traffic.
- **Origin cloaking / VPC Origin** keeps the real origin off the public internet.
- Block malicious requests at the edge before they hit the origin.

→ Later on Wakan: putting WAF + CloudFront in front of S3/API was a natural fit.

### Working with LLMs: context beats "a better model"
- **Tinh Truong:** weak outputs usually come from weak **prompts/context**. A minimal frame: *goal – information – constraints – success criteria*.
- **Duc Dao:** `temperature = 0` does **not** guarantee identical results every run (GPU floating-point noise, API batching). Systems should **tolerate variance**; `temp ≈ 0.1` is often safer than over-trusting zero.

→ For our AI Processor Lambda: force a JSON schema and validate output; never trust raw free text.

### Multi-agent systems and conversational analytics
- **Vy Lam:** credit-scoring for startups with **specialized agents** on Amazon Bedrock inside an isolated VPC — much faster than a manual pipeline.
- **Anh Pham:** **QuickSight Q** demo — natural-language questions over data, less hand-written SQL for simple insights.

### 36-hour prototyping (LotusHacks)
**Mai / Uyen / Thao** shared **UTMorpho**, born from real friction when editing UI with AI. **Smart-diffing** helped cut token usage when the model rewrote UI code.

---

## Personal note

Community Day connected two halves of the problem: **the model** and **the secure platform underneath**. As Wakan's backend lead, I wrote down three design habits:
1. Protect the edge (CloudFront/WAF) before exposing APIs.
2. Structured prompts + JSON validation — do not rely on model luck.
3. Prefer isolated networks (VPC) for sensitive workloads when possible.

### Photos

![Community Day - photo 1](/images/Event1-1.png)


