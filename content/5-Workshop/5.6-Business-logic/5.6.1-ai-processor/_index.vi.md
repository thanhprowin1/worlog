---
title: "Tạo Lambda AI Processor"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.6.1. </b> "
---

#### Mục tiêu

Tạo **Lambda AI Processor** có trách nhiệm:

1. Nhận payload đã validate từ Orchestrator
2. Lấy API key AI từ **AWS Secrets Manager** (không hardcode)
3. Xây **prompt có cấu trúc** từ tùy chọn UI (không phải free-form text)
4. Gọi mô hình AI bên ngoài
5. Ép **JSON schema** cố định trên response
6. Xử lý fallback nếu AI trả cấu trúc sai


#### Bước 1 — Lưu AI API key vào Secrets Manager

1. Mở **AWS Secrets Manager** → **Store a new secret**
2. Cấu hình:
   - **Secret type:** Other type of secret
   - **Key/value:**
     - Key: `api_key`
     - Value: dán API key của provider AI
   - **Secret name:** `wakan/ai-api-key`
   - Encryption: AWS managed key (mặc định) đủ cho lab

<!-- TODO: screenshot - Secrets Manager create secret -->
![Tạo secret](/worlog/images/5-Workshop/5.6-Business-logic/secrets-create1.png)

3. Bấm **Store**

{{% notice important %}}
Không bao giờ ghi API key vào environment variable dạng plain text, không commit lên Git, không log giá trị secret. Luôn lấy qua `GetSecretValue` lúc runtime (hoặc cache ngắn trong memory của container Lambda).
{{% /notice %}}

#### Bước 2 — Tạo IAM Role cho AI Processor

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

Lưu ý: AI Processor **không** cần quyền ghi `wakan-users` hay `wakan-cache` — phần đó thuộc Orchestrator.

<!-- TODO: screenshot - IAM role AI Processor -->


#### Bước 3 — Tạo Lambda function

1. Lambda → **Create function**
2. Cấu hình:
   - **Function name:** `TripAIBackend`
   - **Runtime:** giống Orchestrator (khuyến nghị Python 3.12)
   - **Timeout:** **30 giây** (gọi AI có thể chậm — mặc định 3s quá ngắn)
   - **Memory:** 256 MB hoặc 512 MB
   - **Role:** role AI Processor ở trên

<!-- TODO: screenshot - Create AI Processor function + timeout -->
![Tạo AI Processor](/worlog/images/5-Workshop/5.6-Business-logic/ai-create.png)

#### Bước 4 — Environment variables

| Biến | Giá trị | Ghi chú |
|---|---|---|
| `SECRET_NAME` | `wakan/ai-api-key` | Tên secret trên Secrets Manager |
| `AI_MODEL` | vd. `gpt-4o-mini` hoặc model id | Theo provider |
| `AI_BASE_URL` | vd. `https://api.openai.com/v1` | Hoặc Anthropic / khác |
| `VERIFIED_PLACES_TABLE` | `wakan-verified-places` | Dùng khi plan = pro |

<!-- TODO: screenshot - AI Processor env vars -->


#### Bước 5 — Handler code (đơn giản hóa)

```python
import json, os, boto3, urllib.request, urllib.error
from decimal import Decimal

secrets = boto3.client("secretsmanager")
dynamodb = boto3.resource("dynamodb")
places_table = dynamodb.Table(os.environ.get("VERIFIED_PLACES_TABLE", "wakan-verified-places"))

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
    """Xây prompt có cấu trúc từ tùy chọn UI — không free-form text."""
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
    """Ví dụ gọi OpenAI-compatible chat completions. Chỉnh theo provider."""
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
    content = content.strip()
    if content.startswith("```"):
        content = content.split("\n", 1)[-1]
        if content.endswith("```"):
            content = content.rsplit("```", 1)[0]
    return json.loads(content)


def fallback_itinerary(body):
    """Fallback an toàn khi AI lỗi hoặc JSON sai."""
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
    body = event if isinstance(event, dict) else json.loads(event)
    plan = body.get("_plan", "free")
    city = body.get("city") or "HoChiMinh"

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
        print(json.dumps({"level": "error", "msg": "AI call failed", "error": str(e)[:200]}))
        itinerary = fallback_itinerary(body)

    return {
        "statusCode": 200,
        "body": itinerary
    }
```

<!-- TODO: screenshot - AI Processor code in console -->


#### Bước 6 — Cấp quyền Orchestrator invoke AI Processor

Nếu role Orchestrator chưa có `lambda:InvokeFunction`:

```bash
aws lambda add-permission \
  --function-name wakan-ai-processor \
  --statement-id AllowOrchestratorInvoke \
  --action lambda:InvokeFunction \
  --principal lambda.amazonaws.com \
  --source-arn arn:aws:lambda:ap-southeast-1:<ACCOUNT_ID>:function:wakan-orchestrator \
  --region ap-southeast-1
```

(Hoặc chỉ cần IAM role trên Orchestrator có `lambda:InvokeFunction` khi dùng AWS SDK invoke.)

#### Bước 7 — Unit-test AI Processor (console)

1. Tạo test event:

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

| Mục | Kết quả mong đợi |
|---|---|
| Secret | `wakan/ai-api-key` tồn tại trên Secrets Manager |
| Function | `wakan-ai-processor`, timeout ≥ 30s |
| IAM | Chỉ `GetSecretValue` + query verified places |
| Test Free/Plus | Trả itinerary có cấu trúc |
| Test Pro | Prompt bị giới hạn trong verified places |
| Lỗi AI | Trả fallback, không uncaught 500 |
