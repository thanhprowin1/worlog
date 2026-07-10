---
title: "Gắn WAF (tùy chọn)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

#### Mục tiêu

Gắn **AWS WAF** vào CloudFront distribution để lọc các tấn công web phổ biến (SQL injection, XSS, bot xấu) trước khi traffic tới origin.

{{% notice note %}}
Bước này là **tùy chọn** cho MVP. Bạn có thể bỏ qua và sang Bước 5.4 nếu muốn hoàn thành core flow trước. WAF phát sinh chi phí nhỏ (Web ACL + phí request).
{{% /notice %}}

#### Bước 1 — Tạo Web ACL

1. Mở **AWS WAF & Shield** → **Web ACLs** → **Create web ACL**
2. Cấu hình:
   - **Resource type:** Amazon CloudFront distributions  
     *(WAF cho CloudFront được quản lý ở **us-east-1** dù resource khác của bạn ở Singapore)*
   - **Name:** `TripAI-Web-Firewall`
   - **CloudWatch metric name:** `wakanWebAcl`

<!-- TODO: screenshot - form Create web ACL (resource type CloudFront) -->
![Tạo Web ACL](/images/5-Workshop/5.3-Frontend-edge/waf-create.png)

3. Bấm **Next**

#### Bước 2 — Thêm managed rule groups

Ở trang **Add rules and rule groups**, bấm **Add rules** → **Add managed rule groups**.

Bật các AWS Managed Rules sau (khuyến nghị cho web app public):

| Rule group | Mục đích |
|---|---|
| **AWSManagedRulesCommonRuleSet** | Bảo vệ cốt lõi (XSS, input xấu, …) |
| **AWSManagedRulesKnownBadInputsRuleSet** | Chặn pattern độc hại đã biết |
| **AWSManagedRulesAmazonIpReputationList** | Chặn IP có reputation xấu / bot |

Với mỗi rule group:
- Giữ **Action** theo mặc định của managed rule
- Đảm bảo tổng capacity không vượt giới hạn Web ACL (mặc định 1,500 WCU)

<!-- TODO: screenshot - chọn managed rule groups -->
![Managed rules](/images/5-Workshop/5.3-Frontend-edge/waf-rules.png)

Bấm **Add rules** → **Next**.

#### Bước 3 — Default action & review

1. **Default web ACL action:** Allow  
   *(chỉ request khớp rule Block mới bị chặn)*
2. Review settings → **Create web ACL**

<!-- TODO: screenshot - Web ACL tạo thành công -->
![Web ACL đã tạo](/images/5-Workshop/5.3-Frontend-edge/waf-created.png)

#### Bước 4 — Associate với CloudFront

1. Mở Web ACL vừa tạo → **Associated AWS resources** → **Add AWS resources**
2. Chọn CloudFront distribution của bạn
3. Bấm **Add**

Hoặc từ **CloudFront** → distribution → **Security** → **Edit** → chọn Web ACL `TripAI-Web-Firewall` → Save.

<!-- TODO: screenshot - Web ACL đã associate với CloudFront distribution -->
![WAF associated](/images/5-Workshop/5.3-Frontend-edge/waf-associated.png)

#### Bước 5 — Kiểm tra nhanh

1. Mở CloudFront URL trên trình duyệt — site vẫn load bình thường
2. (Tùy chọn) Vào **WAF → Web ACL → Overview** sau vài request để xem Sampled requests / metrics

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Web ACL tồn tại | Tên `TripAI-Web-Firewall`, resource type CloudFront |
| Managed rules | Ít nhất Common + KnownBadInputs + IP Reputation |
| Association | Đã gắn với CloudFront distribution |
| Site vẫn chạy | Frontend load qua HTTPS như trước |

{{% notice tip %}}
**Gợi ý chi phí:** Nếu chỉ cần WAF cho demo/báo cáo, hãy tạo, chụp screenshot, rồi gỡ association và xóa Web ACL ở bước Cleanup để tránh phí request kéo dài.
{{% /notice %}}
