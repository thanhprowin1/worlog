---
title: "Data Layer with DynamoDB"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Goal

In this section, you will create the DynamoDB tables that store Wakan’s core data:

| Table | Purpose |
|---|---|
| **Users / Subscriptions** | User identity link, plan (Free / Plus / Pro), remaining daily quota |
| **Cache** | Previously generated itineraries, keyed by rounded location + preferences, with TTL |
| **Verified Places** | Curated places for the **Pro** plan so AI does not invent locations |

```
Lambda Orchestrator
    │
    ├── GetItem / UpdateItem  →  wakan-users
    ├── GetItem / PutItem     →  wakan-cache   (TTL)
    └── Query / Scan (Pro)    →  wakan-verified-places
```

{{% notice note %}}
Lambda code that reads/writes these tables is implemented in **Step 5.6**. Here you only create tables, define keys, enable TTL, and insert sample data for testing.
{{% /notice %}}

#### Content

1. [Create Users / Subscriptions table](5.5.1-users/)
2. [Create Cache table with TTL](5.5.2-cache/)
