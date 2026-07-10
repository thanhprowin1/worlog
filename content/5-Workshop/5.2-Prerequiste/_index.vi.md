---
title: "Chuẩn bị"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Yêu cầu hệ thống

- Trình duyệt hiện đại (Chrome, Firefox, Edge)
- Terminal có sẵn `curl` hoặc `wget`
- Tùy chọn: Node.js 18+ nếu bạn cần rebuild frontend tại máy local

#### Tài khoản AWS & Region

- Tài khoản AWS đang hoạt động với Free Tier hoặc credit khả dụng
- Tất cả tài nguyên sẽ được triển khai trong region: **ap-southeast-2 **
  - Gần Việt Nam → độ trễ thấp cho người dùng Đông Nam Á
  - Đầy đủ dịch vụ (Cognito, API Gateway, Lambda, DynamoDB, CloudFront, WAF, Secrets Manager)

<!-- TODO: screenshot - cách đổi region trong console -->
![Chọn region](/images/5-Workshop/5.2-prerequisite/region.png)

#### Quyền IAM

Gắn policy sau vào IAM user/role trước khi bắt đầu workshop này:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "WakanWorkshop",
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket", "s3:PutObject", "s3:PutBucketPolicy", "s3:PutBucketPublicAccessBlock", "s3:PutBucketWebsite", "s3:PutAccelerateConfiguration", "s3:ListBucket", "s3:GetObject", "s3:DeleteObject", "s3:DeleteBucket", "s3:PutBucketAcl", "s3:GetBucketLocation", "s3:ListAllMyBuckets",
                "cloudfront:*",
                "wafv2:*",
                "cognito-idp:CreateUserPool", "cognito-idp:UpdateUserPool", "cognito-idp:CreateUserPoolClient", "cognito-idp:DescribeUserPool", "cognito-idp:DescribeUserPoolClient", "cognito-idp:AdminCreateUser", "cognito-idp:AdminSetUserPassword", "cognito-idp:UpdateUserPoolClient", "cognito-idp:DeleteUserPool", "cognito-idp:DeleteUserPoolClient", "cognito-idp:ListUserPools",
                "apigateway:*",
                "lambda:*",
                "dynamodb:CreateTable", "dynamodb:DeleteTable", "dynamodb:DescribeTable", "dynamodb:UpdateTable", "dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:UpdateItem", "dynamodb:Query", "dynamodb:Scan", "dynamodb:DeleteItem", "dynamodb:TagResource", "dynamodb:ListTables",
                "secretsmanager:CreateSecret", "secretsmanager:PutSecretValue", "secretsmanager:GetSecretValue", "secretsmanager:DeleteSecret", "secretsmanager:DescribeSecret", "secretsmanager:TagResource",
                "iam:CreateRole", "iam:AttachRolePolicy", "iam:PutRolePolicy", "iam:PassRole", "iam:GetRole", "iam:DetachRolePolicy", "iam:DeleteRole", "iam:ListRoles",
                "cloudwatch:CreateLogGroup", "cloudwatch:PutRetentionPolicy", "cloudwatch:DeleteLogGroup", "cloudwatch:PutMetricData", "logs:*",
                "budgets:*"
            ],
            "Resource": "*"
        }
    ]
}
```

<!-- TODO: screenshot - cách attach policy vào IAM user -->
![IAM policy](/images/5-Workshop/5.2-prerequisite/iam-policy.png)

#### API key AI

Bạn cần một API key từ nhà cung cấp AI bên ngoài (ví dụ: OpenAI, Anthropic). Khóa này sẽ được lưu vào **AWS Secrets Manager** trong lab — không bao giờ ghi cứng vào mã nguồn.

- Đăng ký tại nhà cung cấp AI bạn chọn
- Tạo API key
- Lưu ở nơi an toàn — bạn sẽ dán vào Secrets Manager ở bước sau

<!-- TODO: screenshot - nơi tạo API key trên provider -->
![AI API key](/images/5-Workshop/5.2-prerequisite/api-key.png)

#### Frontend đã build sẵn

Một trang web tĩnh đã được build sẵn cho workshop này. Bạn không cần viết code frontend.

- Trang bao gồm trang đăng nhập (Cognito Hosted UI), form chọn tùy chọn chuyến đi và giao diện hiển thị lịch trình
- Tải về và giải nén file zip trước Bước 3

[Tải về frontend assets](#)

#### Công cụ

Cài các công cụ sau trước khi bắt đầu (nếu chưa có):

```bash
# AWS CLI — https://aws.amazon.com/cli/
aws --version

# jq (tùy chọn, để parse JSON trong terminal)
jq --version
```

#### Chi phí & thời gian ước tính

- **Thời gian:** khoảng 2–3 giờ để hoàn thành tất cả các bước
- **Chi phí:** rất thấp trong giai đoạn phát triển; chi phí production phụ thuộc lưu lượng. Sử dụng cảnh báo AWS Budgets (Bước 7) để theo dõi chi tiêu.
