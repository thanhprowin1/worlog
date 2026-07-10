---
title: "Tạo bảng Cache với TTL"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

#### Mục tiêu

Tạo bảng cache lịch trình đã sinh để các request tương tự **không** gọi AI lại. Item hết hạn tự động nhờ DynamoDB **TTL**.

#### Vì sao cache quan trọng với Wakan

- Gọi AI là nguồn chi phí chính
- Nhiều user gần cùng thành phố + cùng tùy chọn có thể dùng chung kết quả
- Cache key dùng **tọa độ làm tròn** (bảo mật riêng tư) + tùy chọn có cấu trúc (ổn định, không phải prompt tự do)

#### Thiết kế bảng

| Thuộc tính | Giá trị |
|---|---|
| **Tên bảng** | `TripAITravelPlan` |
| **Partition key** | `cacheKey` (String) |
| **Sort key** | không (MVP) |
| **Capacity** | On-demand |
| **TTL attribute** | `expiresAt` (Number — Unix epoch **giây**) |

#### Cách tạo `cacheKey` (Orchestrator sẽ implement)

Ví dụ thuật toán (ghi lại cho thống nhất):

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

Chuỗi thô trước khi hash, ví dụ:

```
10.78|106.70|1|low|solo|motorbike|food|near
```

Lab có thể lưu **hash** hoặc chuỗi readable để debug.

#### Attribute gợi ý

| Attribute | Type | Ghi chú |
|---|---|---|
| `cacheKey` | S | PK |
| `itinerary` | S hoặc M | JSON string lịch trình, hoặc Map |
| `planUsed` | S | free / plus / pro (analytics) |
| `createdAt` | S | ISO-8601 |
| `expiresAt` | N | Unix seconds — **trường TTL** |
| `ttlHours` | N | 6 / 12 / 48 |

#### TTL theo loại chuyến (chính sách lab)

| Loại chuyến | TTL gợi ý | Công thức `expiresAt` |
|---|---|---|
| Ngắn / gần | 6–12 giờ | `now + 6*3600` hoặc `12*3600` |
| Nhiều ngày | 24–48 giờ | `now + 24*3600` hoặc `48*3600` |
| Free | 6 giờ | cache ngắn hơn |
| Plus | 12 giờ | |
| Pro | 48 giờ | tái sử dụng lâu hơn |

#### Bước 1 — Tạo bảng

1. DynamoDB → **Create table**
2. Cấu hình:
   - **Table name:** `TripAITravelPlan`
   - **Partition key:** `cacheKey` · **String**
   - **Capacity mode:** On-demand
3. Create table → đợi **Active**

<!-- TODO: screenshot - form Create table wakan-cache -->


#### Bước 2 — Bật TTL

1. Mở bảng `TripAITravelPlan` → **Additional settings** (hoặc **Time to live**)
2. **Enable TTL**
3. **TTL attribute name:** `expiresAt`
4. Confirm / Save

<!-- TODO: screenshot - TTL enabled với attribute expiresAt -->


{{% notice note %}}
DynamoDB TTL xóa item **bất đồng bộ** (có thể trễ sau thời điểm hết hạn). Item hết hạn vẫn có thể xuất hiện ngắn trong scan; Orchestrator nên bỏ qua nếu `expiresAt < now`.
{{% /notice %}}

#### Bước 3 — Chèn cache item mẫu

Tính epoch mẫu (ví dụ 12 giờ sau). Hoặc:

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
    "S": "{\"title\":\"Tour ẩm thực nửa ngày Q1\",\"days\":[{\"day\":1,\"stops\":[{\"name\":\"Chợ Bến Thành\",\"durationHours\":1.5}]}]}"
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

Thay `expiresAt` bằng Unix timestamp **tương lai** thật.

<!-- TODO: screenshot - sample cache item -->


#### Bước 4 — Kiểm tra đọc (tùy chọn)

```bash
aws dynamodb get-item \
  --table-name wakan-cache \
  --key '{"cacheKey":{"S":"lab-demo-10.78-106.70-1-low-solo-near"}}' \
  --region ap-southeast-1
```

#### Logic cache của Orchestrator (preview Bước 5.6)

```
1. Tạo cacheKey từ lat/lng làm tròn + tùy chọn UI
2. GetItem(wakan-cache, cacheKey)
3. Nếu có item VÀ expiresAt > now  →  trả itinerary (cache HIT)
4. Không  →  gọi AI Processor  →  PutItem với expiresAt mới  →  trả (cache MISS)
```

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Bảng | `TripAITravelPlan` **Active** |
| PK | `cacheKey` (String) |
| TTL | Bật trên attribute `expiresAt` |
| Item mẫu | Tồn tại với `expiresAt` trong tương lai |
| Riêng tư | Key dùng tọa độ làm tròn, không lưu GPS chính xác dài hạn |
