---
title: "Xóa IAM, giám sát & kiểm tra cuối"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.8.3. </b> "
---

#### Mục tiêu

Xóa IAM role còn sót, artifact CloudWatch, Budgets (tùy chọn), và chạy checklist kiểm tra cuối.

#### Bước 1 — IAM roles (execution role Lambda)

1. **IAM** → **Roles**
2. Xóa role lab (gỡ policy trước nếu console yêu cầu):
   -  `wakan-ai-processor-role`(hoặc tên bạn dùng)
3. Nếu có custom policy workshop riêng, xóa ở **Policies** khi không còn attach

<!-- TODO: screenshot - IAM roles đã dọn -->
![Xóa IAM roles](/images/5-Workshop/5.8-Cleanup/iam-delete.png)

{{% notice note %}}
Không xóa được role còn gắn Lambda. Hãy xóa function (Bước 5.8.2) trước.
{{% /notice %}}

#### Bước 2 — CloudWatch

| Artifact | Hành động |
|---|---|
| Alarms (`wakan-orchestrator-errors`, …) | Delete alarm |
| Dashboard `Wakan-Overview` | Delete dashboard |
| SNS topic `wakan-alerts` | Xóa subscription → xóa topic |
| Log groups `/aws/lambda/wakan-*` | Xóa log group (tùy chọn nhưng dừng phí lưu trữ) |

1. CloudWatch → **Alarms** → chọn → **Delete**
2. **Dashboards** → xóa `Wakan-Overview`
3. **SNS** → xóa topic `wakan-alerts` (nếu tạo)
4. **Log groups** → xóa `/aws/lambda/TripAIBackend`

<!-- TODO: screenshot - alarms/dashboard/log groups đã dọn -->
![Dọn CloudWatch](/images/5-Workshop/5.8-Cleanup/cw-clean.png)

```bash
aws logs delete-log-group --log-group-name /aws/lambda/wakan-orchestrator --region ap-southeast-1
aws logs delete-log-group --log-group-name /aws/lambda/wakan-ai-processor --region ap-southeast-1
```

#### Bước 3 — AWS Budgets (tùy chọn)

1. **Billing** → **Budgets**
2. Xóa `wakan-monthly-budget` nếu không còn cần alert
3. Hoặc giữ nếu account còn workload khác

<!-- TODO: screenshot - budget đã xóa hoặc giữ kèm ghi chú -->

#### Bước 4 — Checklist kiểm tra cuối

Tìm trên console (hoặc CLI) resource còn sót chữ `wakan`:

| Service | Kiểm tra |
|---|---|
| CloudFront | Không còn distribution wakan |
| S3 | Không còn bucket `wakan-frontend-*` |
| API Gateway | Không còn `wakan-api` |
| Lambda | Không còn function `wakan-*` |
| Cognito | Không còn pool `wakan-users` |
| DynamoDB | Không còn bảng `wakan-*` |
| Secrets Manager | Không còn `wakan/ai-api-key` |
| WAF | Không còn `wakan-web-acl` |
| IAM | Không còn execution role lab |
| CloudWatch | Không còn alarm / log group lab (nếu đã xóa) |

```bash
# Quét nhanh CLI (ví dụ)
aws lambda list-functions --region ap-southeast-1 --query "Functions[?contains(FunctionName, 'wakan')].FunctionName"
aws dynamodb list-tables --region ap-southeast-1 --query "TableNames[?contains(@, 'wakan')]"
aws s3 ls | findstr wakan
```

<!-- TODO: screenshot - CLI/console không còn resource wakan -->


#### Bước 5 — Cost Explorer (ngày hôm sau)

1. Sau khoảng 24 giờ, mở **Cost Explorer**
2. Xác nhận chi phí liên quan Wakan đã phẳng / chỉ residual (CloudFront đôi khi trễ vài giờ)
3. Tiếp tục theo dõi billing AI provider bên ngoài riêng

#### Hoàn thành workshop

Bạn đã đi hết lộ trình hands-on Wakan:

| Bước | Nội dung |
|---|---|
| 5.1 | Tổng quan & kiến trúc |
| 5.2 | Chuẩn bị |
| 5.3 | Frontend & biên (S3, CloudFront, WAF) |
| 5.4 | Auth & API (Cognito, API Gateway, Usage Plans) |
| 5.5 | Lớp dữ liệu (DynamoDB) |
| 5.6 | Nghiệp vụ (Orchestrator + AI Processor) |
| 5.7 | Bảo mật, giám sát & budgets |
| 5.8 | Dọn dẹp |

{{% notice tip %}}
Cho báo cáo thực tập, **giữ screenshot chụp trước khi cleanup**. Sau khi xóa, rất khó chụp lại bằng chứng console nếu không deploy lại.
{{% /notice %}}
