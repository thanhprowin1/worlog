---
title: "Xóa tài nguyên biên & API"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

#### Mục tiêu

Xóa CloudFront, WAF và API Gateway trước — chúng nằm ở biên và phụ thuộc origin phía sau.

#### Bước 1 — CloudFront distribution

1. Mở **CloudFront** → **Distributions** → chọn distribution của bạn
2. Nếu đã gắn WAF: **Security** → gỡ Web ACL → Save
3. **Disable** distribution → đợi status **Deployed** (Disabled)
4. **Delete** distribution

<!-- TODO: screenshot - distribution Disabled rồi Delete -->
![Xóa CloudFront](/images/5-Workshop/5.8-Cleanup/cf-delete.png)

{{% notice note %}}
**Không** xóa được distribution khi vẫn Enabled. Luôn Disable → đợi → Delete.
{{% /notice %}}

CLI gợi ý:

```bash
aws cloudfront get-distribution-config --id E1234567890ABC > /tmp/cf.json
# Set Enabled=false, update với IfMatch ETag, đợi, rồi:
aws cloudfront delete-distribution --id E1234567890ABC --if-match <ETAG>
```

#### Bước 2 — Origin Access Control (OAC)

1. CloudFront → **Origin access** → chọn `wakan-oac`
2. **Delete** nếu không còn distribution nào dùng

<!-- TODO: screenshot - xóa OAC -->
![Xóa OAC](/images/5-Workshop/5.8-Cleanup/oac-delete.png)

#### Bước 3 — WAF Web ACL (nếu đã tạo)

1. Mở **WAF & Shield** (scope **CloudFront** / global, UI thường ở **us-east-1**)
2. Xác nhận **không còn associated resources**
3. Mở Web ACL `TripAI-Web-Firewall` (hoặc tên bạn dùng) → **Delete**

<!-- TODO: screenshot - Web ACL đã xóa / không association -->
![Xóa WAF](/images/5-Workshop/5.8-Cleanup/waf-delete.png)

#### Bước 4 — API Gateway

1. **API Gateway** → **Usage plans**
   - Gỡ stage / API keys
   - Xóa plan: `wakan-plan-free`, `wakan-plan-plus`, `wakan-plan-pro` (nếu có)
2. **API keys** → xóa key lab (`wakan-free-key`, …)
3. **APIs** → chọn `wakan-api` (hoặc `TripAI-API`) → **Actions** → **Delete API**
4. Confirm

<!-- TODO: screenshot - API đã xóa -->
![Xóa API Gateway](/images/5-Workshop/5.8-Cleanup/apigw-delete.png)

#### Checkpoint

| Mục | Kỳ vọng |
|---|---|
| CloudFront | Distribution đã xóa (không chỉ Disabled) |
| OAC | Đã xóa hoặc không còn dùng |
| WAF | Web ACL đã xóa (nếu dùng) |
| API Gateway | API + usage plans + keys đã hết |
