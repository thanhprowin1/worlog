---
title: "Rà soát IAM least privilege"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---

#### Mục tiêu

Rà soát và siết chặt IAM role để mỗi thành phần chỉ có quyền cần thiết. Đây là **best practice bảo mật** — không phải resource mới để tạo.

#### Bảng tóm tắt role

| Role | Gắn cho | Quyền | Cấm |
|---|---|---|---|
| `wakan-orchestrator-role` | Lambda `wakan-orchestrator` | DynamoDB read/write trên `wakan-users` + `wakan-cache`; invoke Lambda trên `wakan-ai-processor` | Không cần |
| `wakan-ai-processor-role` | Lambda `wakan-ai-processor` | Secrets Manager `GetSecretValue` trên `wakan/ai-api-key`; DynamoDB Query trên `wakan-verified-places` | Không ghi vào bảng users/cache |
| `workshop-admin-role` | IAM user/role của bạn | Policy từ Bước 5.2 (tạo + dọn tài nguyên) | N/A |

#### Bước 1 — Xác minh role Orchestrator

1. Mở **IAM** → **Roles** → tìm `wakan-orchestrator-role` (hoặc tên bạn dùng)
2. Kiểm tra **Permissions policy**:

```jsonc
{
  // Chỉ nên chứa các action sau:
  // - logs:CreateLogGroup, logs:CreateLogStream, logs:PutLogEvents
  // - dynamodb:GetItem, dynamodb:UpdateItem, dynamodb:Query   → wakan-users ARN
  // - dynamodb:GetItem, dynamodb:PutItem                       → wakan-cache ARN
  // - lambda:InvokeFunction                                    → wakan-ai-processor ARN
}
```

<!-- TODO: screenshot - Orchestrator IAM policy -->
![IAM Orchestrator](/images/5-Workshop/5.7-Security-monitoring/iam-orchestrator.png)

#### Bước 2 — Xác minh role AI Processor

1. Mở **IAM** → **Roles** → tìm `wakan-ai-processor-role`
2. Kiểm tra **Permissions policy**:

```jsonc
{
  // Chỉ nên chứa các action sau:
  // - logs:*Log*
  // - secretsmanager:GetSecretValue    → wakan/ai-api-key ARN
  // - dynamodb:Query, dynamodb:GetItem → wakan-verified-places ARN
}
```

Xác nhận nó **không** có các quyền này:
- ❌ `dynamodb:PutItem` trên `wakan-users` hoặc `wakan-cache`
- ❌ `secretsmanager:CreateSecret`, `DeleteSecret` (chỉ `GetSecretValue`)

<!-- TODO: screenshot - AI Processor IAM policy -->


#### Bước 3 — Thêm deny xóa (tùy chọn nhưng khuyến nghị)

Để tránh vô tình xóa trong thời gian workshop, thêm **SCP** hoặc **IAM policy** cấm các hành động phá hủy trừ khi được nhóm cụ thể ủy quyền (vd. `workshop-admins`).

Ví dụ inline policy cho user không phải admin:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PreventDestructiveActions",
      "Effect": "Deny",
      "Action": [
        "dynamodb:DeleteTable",
        "lambda:DeleteFunction",
        "apigateway:DELETE",
        "cloudfront:DeleteDistribution"
      ],
      "Resource": "*"
    }
  ]
}
```

Điều này buộc hành động có chủ đích trước khi xóa resource — hữu ích khi demo hoặc workshop nhóm.

<!-- TODO: screenshot - SCP / deny policy đã gắn -->


#### Bước 4 — Ghi chú xoay secret (production)

Với production, nên xoay API key định kỳ và lưu dưới dạng versioned secret:

```bash
# Ví dụ: xoay bằng cách tạo version mới với label mới
aws secretsmanager put-secret-value \
  --secret-id wakan/ai-api-key \
  --secret-string '{"api_key":"KEY_MỚI"}' \
  --version-stages ["AWSCURRENT"] \
  --region ap-southeast-1
```

Giữ version cũ ít nhất 30 ngày trước khi xóa (`AWSPREVIOUS`).

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Role Orchestrator | Chỉ đọc/ghi DynamoDB + invoke AI Processor |
| Role AI Processor | Chỉ đọc secret + query verified places; không ghi users/cache |
| Deny policy | Hành động phá hủy bị chặn với user không phải admin |
| Xoay secret | Đã ghi rõ quy trình |
