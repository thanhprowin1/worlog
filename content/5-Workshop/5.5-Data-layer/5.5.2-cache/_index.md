---
title: "Create Cache table with TTL"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

#### Objective

Create a cache table for generated itineraries so repeated similar requests do **not** call the AI model again. Items expire automatically via DynamoDB **TTL**.

#### Why cache matters for Wakan

- AI calls are the main cost driver
- Many users near the same city with the same options can share a result
- Cache key uses **rounded coordinates** (privacy) + structured preferences (stable, not free-form prompts)

#### Table design

| Property | Value |
|---|---|
| **Table name** | `TripAITravelPlan` |
| **Partition key** | `cacheKey` (String) |
| **Sort key** | none (MVP) |
| **Capacity** | On-demand |
| **TTL attribute** | `expiresAt` (Number — Unix epoch **seconds**) |

#### How to build `cacheKey` (Orchestrator will implement this)

Example algorithm (document for consistency):

```
cacheKey = sha256(
  round(lat, 2) + "|" +
  round(lng, 2) + "|" +
  days + "|" +
  budget + "|" +
  companions + "|" +
  transport + "|" +
  experienceType + "|" +
  distanceScope   // near | far
)
```

Example raw string before hash:

```
10.78|106.70|1|low|solo|motorbike|food|near
```

Store either the **hash** as `cacheKey`, or a readable joined string for lab debugging.

#### Suggested item attributes

| Attribute | Type | Notes |
|---|---|---|
| `cacheKey` | S | PK |
| `itinerary` | S or M | JSON string of itinerary, or Map |
| `planUsed` | S | free / plus / pro (optional analytics) |
| `createdAt` | S | ISO-8601 |
| `expiresAt` | N | Unix seconds — **TTL field** |
| `ttlHours` | N | 6 / 12 / 48 for documentation |

#### TTL by trip type (lab policy)

| Trip type | Suggested TTL | `expiresAt` formula |
|---|---|---|
| Short / nearby | 6–12 hours | `now + 6*3600` or `12*3600` |
| Multi-day | 24–48 hours | `now + 24*3600` or `48*3600` |
| Free plan | 6 hours | shorter cache |
| Plus | 12 hours | |
| Pro | 48 hours | longer reuse |

#### Step 1 — Create the table

1. DynamoDB → **Create table**
2. Configure:
   - **Table name:** `TripAITravelPlan`
   - **Partition key:** `cacheKey` · **String**
   - **Capacity mode:** On-demand
3. Create table → wait **Active**

<!-- TODO: screenshot - form Create table wakan-cache -->


#### Step 2 — Enable TTL

1. Open table `TripAITravelPlan` → **Additional settings** (or **Actions** → **Time to live**)
2. **Enable TTL**
3. **TTL attribute name:** `expiresAt`
4. Confirm / Save

<!-- TODO: screenshot - TTL enabled với attribute expiresAt -->


{{% notice note %}}
DynamoDB TTL deletes items **asynchronously** (usually within ~48 hours of expiry, often sooner). Expired items may still appear briefly in scans; Orchestrator should also ignore items where `expiresAt < now`.
{{% /notice %}}

#### Step 3 — Insert a sample cache item

Calculate a sample epoch (example: 12 hours from now). Or use:

```bash
# Linux / Git Bash / WSL
date -d '+12 hours' +%s

# PowerShell
[int][double]::Parse((Get-Date).AddHours(12).ToUniversalTime().Subtract([datetime]'1970-01-01').TotalSeconds)
```

Create item (JSON):

```json
{
  "cacheKey": {
    "S": "lab-demo-10.78-106.70-1-low-solo-near"
  },
  "itinerary": {
    "S": "{\"title\":\"Half-day food tour in D1\",\"days\":[{\"day\":1,\"stops\":[{\"name\":\"Ben Thanh Market\",\"durationHours\":1.5}]}]}"
  },
  "planUsed": {
    "S": "free"
  },
  "createdAt": {
    "S": "2026-07-09T12:00:00Z"
  },
  "expiresAt": {
    "N": "1752105600"
  },
  "ttlHours": {
    "N": "12"
  }
}
```

Replace `expiresAt` with a real future Unix timestamp.

<!-- TODO: screenshot - sample cache item -->


#### Step 4 — Verify read path (optional)

```bash
aws dynamodb get-item \
  --table-name wakan-cache \
  --key '{"cacheKey":{"S":"lab-demo-10.78-106.70-1-low-solo-near"}}' \
  --region ap-southeast-1
```

#### Orchestrator cache logic (preview for Step 5.6)

```
1. Build cacheKey from rounded lat/lng + UI options
2. GetItem(wakan-cache, cacheKey)
3. If item exists AND expiresAt > now  →  return itinerary (cache HIT)
4. Else  →  call AI Processor  →  PutItem with new expiresAt  →  return (cache MISS)
```

#### Checkpoint

| Item | Expected result |
|---|---|
| Table | `TripAITravelPlan` **Active** |
| PK | `cacheKey` (String) |
| TTL | Enabled on attribute `expiresAt` |
| Sample item | Exists with future `expiresAt` |
| Privacy | Keys use rounded coords, not precise GPS long-term |
