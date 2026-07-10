---
title: "Week 10 Worklog"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:
* Debug and optimize the cache hit/miss flow, tune TTL based on trip length (near/far/multi-day).
* Participate in the first full end-to-end system test and cross-team bug-fixing session.

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Status |
| --- | --- | --- | --- | --- |
| 2 | - First end-to-end test: login → select options → receive itinerary <br> - Logged 3 issues: Lambda timeout, cache key mismatch, response format drift | 22/06/2026 | 22/06/2026 | Completed |
| 3 | - Debug cache key: normalize hash string, add fallback for payloads missing optional params <br> - Lambda AI Processor timeout from 3s to 10s | 23/06/2026 | 23/06/2026 | Completed |
| 4 | - Tune TTL by trip type: short (1–2 days) → 6h, medium (3–5 days) → 12h, long (>5 days) → 24h <br> - Verify TTL expiry works correctly via DynamoDB console | 24/06/2026 | 24/06/2026 | Completed |
| 5 | - Fix response format: standardize Orchestrator JSON output to match the API contract locked in Week 8 <br> - Add a validation layer before returning to the client | 25/06/2026 | 25/06/2026 | Completed |
| 6 | - Cross-team bug-fix meeting: redistribute remaining tasks, update timeline | 26/06/2026 | 26/06/2026 | Completed |

### Week 10 Achievements:
* **Stable caching:** Tiered TTL by trip length balances freshness against AI call cost — cache hit rate rose from ~30% to ~65%.
* **Full E2E pass:** After two debug cycles, the system runs smoothly across all three service tiers; no more timeouts or format mismatches.