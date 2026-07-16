---
title: "Create Lambda AI Processor"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.6.1. </b> "
---

#### Objective

Create the **Lambda AI Processor** that:

1. Receives a validated payload from the Orchestrator
2. Fetches the AI API key from **AWS Secrets Manager** (never hardcode secrets)
3. Builds a **structured prompt** from UI options (not free-form user text)
4. Calls the external AI model
5. Enforces a fixed **JSON schema** on the response
6. Handles fallback if AI returns invalid structure


#### Step 1 — Store AI API key in Secrets Manager

1. Open **AWS Secrets Manager** → **Store a new secret**
2. Configure:
   - **Secret type:** Other type of secret
   - **Key/value:**
     - Key: `api_key`
     - Value: paste your AI provider API key
   - **Secret name:** `wakan/ai-api-key`
   - Encryption: AWS managed key (default) is fine for lab



3. Click **Store**

{{% notice important %}}
Never put the API key in Lambda environment variables as plain text, never commit it to Git, and never log the secret value. Always fetch via `GetSecretValue` at runtime (or cache briefly in memory within the same Lambda container).
{{% /notice %}}

#### Step 2 — Create IAM Role for AI Processor

Least privilege:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:ap-southeast-1:<ACCOUNT_ID>:secret:wakan/ai-api-key*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:Query",
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/wakan-verified-places"
    }
  ]
}
```

Note: AI Processor does **not** need write access to `wakan-users` or `wakan-cache` — that stays with Orchestrator.

<!-- TODO: screenshot - IAM role AI Processor -->


#### Step 3 — Create Lambda function

1. Lambda → **Create function**
2. Configure:
   - **Function name:** `TripAIBackend`
   - **Runtime:** same as Orchestrator (Python 3.12 recommended)
   - **Timeout:** **30 seconds** (AI calls can be slow — default 3s is too short)
   - **Memory:** 256 MB or 512 MB
   - **Role:** AI Processor role above

<!-- TODO: screenshot - Create AI Processor function + timeout -->
![Create AI Processor](/worlog/images/5-Workshop/5.6-Business-logic/ai-create.png)

#### Step 4 — Environment variables

| Variable | Value | Notes |
|---|---|---|
| `SECRET_NAME` | `wakan/ai-api-key` | Secrets Manager secret name |
| `AI_MODEL` | e.g. `gpt-4o-mini` or your model id | Provider-specific |
| `AI_BASE_URL` | e.g. `https://api.openai.com/v1` | Or Anthropic / other |
| `VERIFIED_PLACES_TABLE` | `wakan-verified-places` | Used when plan is pro |

<!-- TODO: screenshot - AI Processor env vars -->

#### Step 5 — Handler code (simplified)

```python
import json, os, boto3, urllib.request, urllib.error
from decimal import Decimal

secrets = boto3.client("secretsmanager")
dynamodb = boto3.resource("dynamodb")
places_table = dynamodb.Table(os.environ.get("VERIFIED_PLACES_TABLE", "wakan-verified-places"))

# Simple in-memory cache of the secret for warm containers
_cached_key = None


def get_api_key():
    global _cached_key
    if _cached_key:
        return _cached_key
    resp = secrets.get_secret_value(SecretId=os.environ["SECRET_NAME"])
    secret = json.loads(resp["SecretString"]) if resp["SecretString"].startswith("{") else {"api_key": resp["SecretString"]}
    _cached_key = secret.get("api_key") or secret.get("API_KEY")
    return _cached_key


def query_verified_places(city):
    resp = places_table.query(
        KeyConditionExpression="city = :c",
        ExpressionAttributeValues={":c": city}
    )
    return resp.get("Items", [])


def build_prompt(body, plan, verified_places=None):
    """Build a structured prompt from UI options — never free-form user text."""
    lines = [
        "You are Wakan, a Vietnam travel itinerary assistant.",
        "Generate a travel itinerary as STRICT JSON only (no markdown).",
        f"Days: {body.get('days')}",
        f"Budget: {body.get('budget')}",
        f"Companions: {body.get('companions')}",
        f"Transport: {body.get('transport')}",
        f"Experience type: {body.get('experienceType')}",
        f"Distance scope: {body.get('distanceScope')}",
        f"Location (approx): lat={body.get('lat')}, lng={body.get('lng')}",
        "",
        "Output JSON schema:",
        '{',
        '  "title": "string",',
        '  "days": [',
        '    {',
        '      "day": 1,',
        '      "stops": [',
        '        {"name": "string", "durationHours": 1.5, "placeId": "optional", "notes": "string"}',
        '      ]',
        '    }',
        '  ]',
        '}',
    ]
    if plan == "pro" and verified_places:
        places_json = json.dumps([
            {
                "placeId": p.get("placeId"),
                "name": p.get("name"),
                "category": p.get("category"),
                "budgetLevel": p.get("budgetLevel"),
                "avgDurationHours": float(p.get("avgDurationHours", 1)) if isinstance(p.get("avgDurationHours"), Decimal) else p.get("avgDurationHours"),
            }
            for p in verified_places
        ], ensure_ascii=False)
        lines.append("")
        lines.append("CRITICAL: Only choose places from this verified list. Do NOT invent new places.")
        lines.append(places_json)
        lines.append("Each stop must include placeId from the list.")
    return "\n".join(lines)


def call_ai(prompt, api_key):
    """Example OpenAI-compatible chat completions call. Adjust for your provider."""
    url = os.environ.get("AI_BASE_URL", "https://api.openai.com/v1") + "/chat/completions"
    payload = {
        "model": os.environ.get("AI_MODEL", "gpt-4o-mini"),
        "messages": [
            {"role": "system", "content": "Return valid JSON only."},
            {"role": "user", "content": prompt},
        ],
        "temperature": 0.4,
    }
    req = urllib.request.Request(
        url,
        data=json.dumps(payload).encode("utf-8"),
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {api_key}",
        },
        method="POST",
    )
    with urllib.request.urlopen(req, timeout=25) as resp:
        data = json.loads(resp.read().decode("utf-8"))
    content = data["choices"][0]["message"]["content"]
    # Strip markdown fences if model wraps JSON
    content = content.strip()
    if content.startswith("```"):
        content = content.split("\n", 1)[-1]
        if content.endswith("```"):
            content = content.rsplit("```", 1)[0]
    return json.loads(content)


def fallback_itinerary(body):
    """Safe fallback when AI fails or returns invalid JSON."""
    return {
        "title": "Fallback short itinerary",
        "days": [
            {
                "day": 1,
                "stops": [
                    {
                        "name": "Local market near you",
                        "durationHours": 1.5,
                        "notes": "AI temporarily unavailable — try again later."
                    }
                ]
            }
        ],
        "fallback": True
    }


def validate_schema(itinerary):
    if not isinstance(itinerary, dict):
        return False
    if "title" not in itinerary or "days" not in itinerary:
        return False
    if not isinstance(itinerary["days"], list) or len(itinerary["days"]) == 0:
        return False
    return True


def lambda_handler(event, context):
    # event is the payload from Orchestrator (dict), not API Gateway proxy format
    body = event if isinstance(event, dict) else json.loads(event)
    plan = body.get("_plan", "free")
    city = body.get("city") or "HoChiMinh"  # lab default; production: reverse-geocode rounded coords

    verified = []
    if plan == "pro":
        verified = query_verified_places(city)

    try:
        api_key = get_api_key()
        prompt = build_prompt(body, plan, verified if plan == "pro" else None)
        itinerary = call_ai(prompt, api_key)
        if not validate_schema(itinerary):
            itinerary = fallback_itinerary(body)
    except Exception as e:
        # Do not log api_key or full prompt with secrets
        print(json.dumps({"level": "error", "msg": "AI call failed", "error": str(e)[:200]}))
        itinerary = fallback_itinerary(body)

    return {
        "statusCode": 200,
        "body": itinerary
    }
```

<!-- TODO: screenshot - AI Processor code in console -->


#### Step 6 — Grant Orchestrator permission to invoke AI Processor

If not already covered by the Orchestrator role:

1. Open function `wakan-ai-processor` → **Configuration** → **Permissions**
2. Or add resource-based policy:

```bash
aws lambda add-permission \
  --function-name wakan-ai-processor \
  --statement-id AllowOrchestratorInvoke \
  --action lambda:InvokeFunction \
  --principal lambda.amazonaws.com \
  --source-arn arn:aws:lambda:ap-southeast-1:<ACCOUNT_ID>:function:wakan-orchestrator \
  --region ap-southeast-1
```

(Alternatively, the IAM role on Orchestrator with `lambda:InvokeFunction` is enough when using AWS SDK invoke.)

#### Step 7 — Unit-test AI Processor (console)

1. Create a test event:

```json
{
  "lat": 10.78,
  "lng": 106.70,
  "days": 1,
  "budget": "low",
  "companions": "solo",
  "transport": "motorbike",
  "experienceType": "food",
  "distanceScope": "near",
  "city": "HoChiMinh",
  "_plan": "free",
  "_userId": "test-user"
}
```


#### Checkpoint

| Item | Expected result |
|---|---|
| Secret | `wakan/ai-api-key` exists in Secrets Manager |
| Function | `wakan-ai-processor`, timeout ≥ 30s |
| IAM | Can `GetSecretValue` + query verified places only |
| Free/Plus test | Returns structured itinerary |
| Pro test | Prompt constrained to verified places |
| Failure | Fallback itinerary returned, not uncaught 500 |
