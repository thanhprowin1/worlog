---
title: "Xóa compute, dữ liệu & secrets"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---

#### Mục tiêu

Xóa Lambda, Cognito, bảng DynamoDB, secret Secrets Manager và S3 bucket frontend.

#### Bước 1 — Lambda functions

1. Mở **Lambda** → **Functions**
2. Xóa theo thứ tự (tùy chọn nhưng rõ ràng):
   1. `TripAIBackend` (callee)
3. Confirm từng lần xóa

<!-- TODO: screenshot - danh sách Lambda không còn wakan-* -->
![Xóa Lambda](/worlog/images/5-Workshop/5.8-Cleanup/lambda-delete.png)

```bash
aws lambda delete-function --function-name wakan-orchestrator --region ap-southeast-1
aws lambda delete-function --function-name wakan-ai-processor --region ap-southeast-1
```

#### Bước 2 — Cognito User Pool

1. **Amazon Cognito** → **User pools** → `User pool - weu7cv` (hoặc tên pool của bạn)
2. **App integration** → **Domain** → xóa Cognito domain trước (nếu có)
3. **Delete user pool** → gõ tên để confirm

<!-- TODO: screenshot - user pool đã xóa -->
![Xóa Cognito](/worlog/images/5-Workshop/5.8-Cleanup/cognito-delete.png)

{{% notice note %}}
Có thể cần xóa **domain** trước khi xóa pool. App client sẽ bị xóa cùng pool.
{{% /notice %}}

#### Bước 3 — Bảng DynamoDB

Xóa cả 3 bảng lab:

| Bảng |
|---|
| `wakan-users` |
| `wakan-cache` |
| `wakan-verified-places` |

1. DynamoDB → **Tables** → chọn bảng → **Delete**
2. Gõ tên bảng để confirm
3. Lặp cho từng bảng

<!-- TODO: screenshot - danh sách không còn wakan-* tables -->


```bash
aws dynamodb delete-table --table-name wakan-users --region ap-southeast-2
aws dynamodb delete-table --table-name wakan-cache --region ap-southeast-2
aws dynamodb delete-table --table-name wakan-verified-places --region ap-southeast-2
```

#### Bước 4 — Secrets Manager

1. **Secrets Manager** → secret `wakan/ai-api-key`
2. **Actions** → **Delete secret**
3. Lab: chọn **Force delete without recovery** (hoặc recovery window 7 ngày)

<!-- TODO: screenshot - secret đã xóa / pending deletion -->
![Xóa secret](/worlog/images/5-Workshop/5.8-Cleanup/secrets-delete.png)

```bash
aws secretsmanager delete-secret \
  --secret-id wakan/ai-api-key \
  --force-delete-without-recovery \
  --region ap-southeast-1
```

#### Bước 5 — S3 bucket

Bucket phải **rỗng** trước khi xóa.

1. Mở bucket `waka.id.vn` (hoặc tên bạn dùng)
2. Chọn tất cả object → **Delete** (xóa hết, kể cả version nếu có versioning)
3. Bucket → **Delete** → gõ tên bucket

<!-- TODO: screenshot - empty bucket rồi delete -->
![Xóa S3](/worlog/images/5-Workshop/5.8-Cleanup/s3-delete.png)

```bash
# Cẩn thận — không hoàn tác
aws s3 rm s3://waka.id.vn -<your-unique-id>/ --recursive --region ap-southeast-2
```

#### Checkpoint

| Mục | Kỳ vọng |
|---|---|
| Lambda | Không còn  `wakan-ai-processor` |
| Cognito | User pool + domain đã xóa |
| DynamoDB | Cả 3 bảng đã xóa |
| Secrets | `wakan/ai-api-key` đã xóa hoặc pending |
| S3 | Frontend bucket đã xóa |
