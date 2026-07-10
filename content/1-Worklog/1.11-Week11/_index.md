---
title: "Week 11 Worklog"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:
* Run a light load test, tune Lambda configuration (memory/timeout), and perform a final review of the full business flow.
* Join the team review meeting on testing outcomes before locking the demo build.

### Tasks completed this week:
| Day | Task | Start Date | Completion Date | Status |
| --- | --- | --- | --- | --- |
| 2 | - Team review of testing outcomes <br> - Log optimization targets: AI call latency, Lambda cold start, memory usage | 29/06/2026 | 29/06/2026 | Completed |
| 3 | - Light load test: 20 concurrent requests through API Gateway, measure p50/p95 latency <br> - Found Orchestrator cold start ~1.2s; AI Processor ~2.5s | 30/06/2026 | 30/06/2026 | Completed |
| 4 | - Lambda AI Processor memory from 128MB to 256MB (duration down ~30%) <br> - Raise AI Processor memory from 256MB to 512MB; set timeout to 15s | 01/07/2026 | 01/07/2026 | Completed |
| 5 | - Check edge cases: user with no Users record → auto-init; cache key collision | 02/07/2026 | 02/07/2026 | Completed |
| 6 | - Patch 2 remaining bugs: race condition when two requests increment usedToday simultaneously <br> - Switch to DynamoDB UpdateItem atomic counter instead of read-modify-write | 03/07/2026 | 03/07/2026 | Completed |

### Week 11 Achievements:
* **Stable performance:** After memory/timeout tuning, p95 latency dropped from ~4.5s to ~2.8s; cold start is no longer the main bottleneck.
* **Solid business logic:** Edge cases covered, race condition fixed with atomic counters — ready for the demo.