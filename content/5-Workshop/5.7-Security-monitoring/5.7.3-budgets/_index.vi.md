---
title: "Cảnh báo AWS Budgets"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.7.3. </b> "
---

#### Mục tiêu

Tạo **AWS Budget** kèm email cảnh báo để biết khi chi tiêu tháng sắp chạm ngưỡng. Gọi AI (provider ngoài) cộng chi phí AWS (Lambda, API Gateway, CloudFront, DynamoDB) có thể tăng nhanh khi tải cao.

{{% notice note %}}
AWS Budgets theo dõi **chi phí dịch vụ AWS**. Hóa đơn AI provider ngoài (OpenAI, Anthropic, …) **nằm ngoài** AWS — theo dõi riêng trên dashboard provider. Cache + quota trong Wakan là đòn bẩy chính cho chi phí AI.
{{% /notice %}}

#### Đòn bẩy chi phí đã có trong kiến trúc

| Đòn bẩy | Nơi | Hiệu quả |
|---|---|---|
| Cache HIT | DynamoDB + Orchestrator | Tránh gọi AI hoàn toàn |
| Usage Plan throttle | API Gateway | Giới hạn tốc độ request |
| WAF | CloudFront | Chặn bot spam trước khi tới Lambda |
| Timeout / fallback | AI Processor | Giới hạn call treo tốn kém |

#### Bước 1 — Tạo budget chi phí tháng

1. Mở **AWS Billing** → **Budgets** → **Create budget**
2. Chọn **Use a template** hoặc **Customize**
3. Cấu hình lab khuyến nghị:

| Setting | Giá trị |
|---|---|
| **Budget type** | Cost budget |
| **Name** | `wakan-monthly-budget` |
| **Period** | Monthly |
| **Budget amount** | vd. `$10` (lab) hoặc `$50` (demo) |
| **Budget scope** | All AWS services (hoặc filter theo tag nếu đã tag resource) |

<!-- TODO: screenshot - Create budget form -->
![Tạo budget](/images/5-Workshop/5.7-Security-monitoring/budget-create.png)

#### Bước 2 — Cấu hình alert

Thêm ngưỡng:

| Ngưỡng | Trigger | Hành động |
|---|---|---|
| **50%** budget | Actual cost | Email cảnh báo |
| **80%** budget | Actual cost | Email khẩn |
| **100%** budget | Actual cost | Email critical |
| **100%** budget | Forecasted cost | Email cảnh báo sớm |

1. Nhập email cho từng alert
2. Create budget

<!-- TODO: screenshot - alert thresholds 50/80/100 -->
![Budget alerts](/images/5-Workshop/5.7-Security-monitoring/budget-alerts.png)

#### Bước 3 — (Tùy chọn) Tag resource cho budget lọc

Nếu muốn budget **chỉ cho Wakan**:

1. Tag mọi resource Wakan:
   - Key: `Project`
   - Value: `Wakan`
2. Tạo budget filter theo tag đó

```bash
# Ví dụ: tag Lambda
aws lambda tag-resource \
  --resource arn:aws:lambda:ap-southeast-1:<ACCOUNT_ID>:function:wakan-orchestrator \
  --tags Project=Wakan \
  --region ap-southeast-1
```

Tag tương tự S3, CloudFront, DynamoDB, API Gateway, Secrets, WAF để phân bổ chi phí chính xác.

<!-- TODO: screenshot - resource tags Project=Wakan -->


#### Bước 4 — Confirm SNS / email (nếu cần)

Một số notification budget dùng SNS. Confirm email subscription để alert không kẹt “Pending confirmation”.

#### Bước 5 — Ghi chú giám sát AI ngoài AWS

Thêm đoạn ngắn trong báo cáo:

| Provider | Nơi theo dõi | Hành động gợi ý |
|---|---|---|
| OpenAI / Anthropic / khác | Dashboard billing provider | Bật hard cap tháng nếu có |
| AWS | Budgets + Cost Explorer | `wakan-monthly-budget` |

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Budget tồn tại | Tên `wakan-monthly-budget`, chu kỳ monthly |
| Alerts | Ít nhất 50% và 100% email |
| Email confirmed | Đã confirm subscription nếu có |
| Tags (tùy chọn) | `Project=Wakan` trên resource cốt lõi |
| AI ngoài | Đã ghi rõ theo dõi tách với AWS Budgets |

#### Tóm tắt phần 5.7

| Lĩnh vực | Bạn đã làm |
|---|---|
| **Bảo mật** | Rà soát IAM least privilege; deny-delete tùy chọn; ghi chú xoay secret |
| **Giám sát** | Retention log, alarm lỗi, metric cache tùy chọn, dashboard |
| **Chi phí** | Budget tháng + email đa ngưỡng |

Tiếp theo: **Bước 5.8 Cleanup** — xóa resource theo thứ tự an toàn để dừng mọi chi phí.
