---
title: "Giám sát CloudWatch"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

#### Mục tiêu

Thiết lập retention log CloudWatch, các metric quan trọng và dashboard đơn giản để quan sát:

- Số request API vào hệ thống
- Tỉ lệ cache hit / cache miss
- Lỗi và thời gian chạy Lambda
- Lỗi gọi AI

#### Metric quan trọng với Wakan

| Metric | Nguồn | Vì sao quan trọng |
|---|---|---|
| `API 4XXError` / `5XXError` | API Gateway | Lỗi auth / lỗi server |
| `Count` (requests) | API Gateway | Lưu lượng |
| `Invocations` | Lambda Orchestrator | Tổng request |
| `Errors` | Lambda Orchestrator / AI Processor | Lỗi runtime |
| `Duration` | Lambda AI Processor | Latency AI (chi phí + UX) |
| `Throttles` | Lambda / API Gateway | Áp lực capacity |
| Cache HIT / MISS | Custom metric (tùy chọn) | Hiệu quả kiểm soát chi phí AI |
| `remainingQuota` = 0 | Custom / DynamoDB stream (nâng cao) | Trải nghiệm user |

#### Bước 1 — Đặt retention log

Mặc định CloudWatch Logs không hết hạn (chi phí tích lũy). Đặt retention:

1. Mở **CloudWatch** → **Log groups**
2. Chọn `/aws/lambda/wakan-orchestrator` → **Actions** → **Edit retention setting**
3. Đặt **7 days** (lab) hoặc **30 days** (demo)
4. Làm tương tự cho `/aws/lambda/wakan-ai-processor`

<!-- TODO: screenshot - log retention set to 7 days -->
![Log retention](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-retention.png)

CLI:

```bash
aws logs put-retention-policy \
  --log-group-name /aws/lambda/wakan-orchestrator \
  --retention-in-days 7 \
  --region ap-southeast-1

aws logs put-retention-policy \
  --log-group-name /aws/lambda/wakan-ai-processor \
  --retention-in-days 7 \
  --region ap-southeast-1
```

#### Bước 2 — Tạo CloudWatch Alarm cho lỗi Lambda

1. CloudWatch → **Alarms** → **Create alarm**
2. Chọn metric:
   - Namespace: `AWS/Lambda`
   - Metric name: `Errors`
   - FunctionName: `wakan-orchestrator`
3. Cấu hình:
   - Statistic: Sum
   - Period: 5 minutes
   - Threshold: ≥ 1 (hoặc 5 để ít ồn)
4. Notification:
   - Tạo SNS topic `wakan-alerts`
   - Subscribe email của bạn
5. Alarm name: `wakan-orchestrator-errors`

<!-- TODO: screenshot - alarm create form -->
![Tạo alarm](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-alarm.png)

6. Confirm email SNS subscription

Tùy chọn: lặp lại cho Errors của `wakan-ai-processor` và API Gateway `5XXError`.

#### Bước 3 — Emit custom metric cache (tùy chọn nhưng khuyến nghị)

Trong code Orchestrator, sau cache HIT / MISS, thêm:

```python
import boto3
cloudwatch = boto3.client("cloudwatch")

def put_cache_metric(hit: bool):
    cloudwatch.put_metric_data(
        Namespace="Wakan",
        MetricData=[
            {
                "MetricName": "CacheHit" if hit else "CacheMiss",
                "Value": 1,
                "Unit": "Count",
            }
        ],
    )
```

Gọi `put_cache_metric(True)` khi HIT và `put_cache_metric(False)` khi MISS.

Cấp quyền cho role Orchestrator:

```json
{
  "Effect": "Allow",
  "Action": ["cloudwatch:PutMetricData"],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "cloudwatch:namespace": "Wakan"
    }
  }
}
```



#### Bước 4 — Tạo dashboard đơn giản

1. CloudWatch → **Dashboards** → **Create dashboard** → tên `Giai_doan_1`
2. Thêm widget:

| Widget | Type | Nội dung |
|---|---|---|
| API requests | Line | API Gateway `Count` cho `wakan-api` / stage `prod` |
| Lambda errors | Number | Errors Orchestrator + AI Processor |
| AI duration | Line | Duration AI Processor p50 / p99 |
| Cache HIT vs MISS | Stacked | Custom `Wakan/CacheHit` và `Wakan/CacheMiss` |

3. Save dashboard

<!-- TODO: screenshot - Wakan-Overview dashboard -->
![CloudWatch dashboard](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-dashboard.png)

#### Bước 5 — Kiểm tra log mẫu

1. Gửi vài request test (Bước 5.6.3)
2. Mở log group → stream mới nhất
3. Xác nhận log có cấu trúc dạng:

```json
{"level":"info","msg":"cache_hit","cacheKey":"a1b2c3...","userId":"sub-xxx"}
{"level":"error","msg":"AI call failed","error":"timeout"}
```

{{% notice important %}}
Không bao giờ log: API key, header Authorization đầy đủ, tọa độ GPS chính xác, hoặc raw prompt có thể chứa dữ liệu nhạy cảm. Chỉ log tọa độ làm tròn và các option enum.
{{% /notice %}}

<!-- TODO: screenshot - clean structured log (no secrets) -->
![Log sạch](/worlog/images/5-Workshop/5.7-Security-monitoring/cw-logs-clean.png)

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Log retention | 7 hoặc 30 ngày cho cả 2 log group Lambda |
| Alarm | Ít nhất Errors Orchestrator → email SNS |
| Custom metrics (tùy chọn) | Xuất hiện `Wakan/CacheHit` và `Wakan/CacheMiss` |
| Dashboard | `Wakan-Overview` hiện traffic + errors |
| Log hygiene | Không có secret hay GPS chính xác trong log |
