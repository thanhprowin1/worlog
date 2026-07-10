---
title: "Prerequisites"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### System requirements

- A modern browser (Chrome, Firefox, Edge)
- Terminal with `curl` or `wget` available
- Optional: Node.js 18+ if you need to rebuild the frontend locally

#### AWS account & region

- An active AWS account with Free Tier or available credits
- All resources will be deployed in region: **ap-southeast-2 **
  - Closest to Vietnam → lower latency for users in Southeast Asia
  - Full service availability (Cognito, API Gateway, Lambda, DynamoDB, CloudFront, WAF, Secrets Manager)

<!-- TODO: screenshot - cách đổi region trong console -->
![Region selection](/images/5-Workshop/5.2-prerequisite/region.png)

#### IAM permissions

Attach the following policy to your IAM user/role before starting this workshop:

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

#### AI API key

You will need an API key from an external AI provider (e.g., OpenAI, Anthropic). This key will be stored in **AWS Secrets Manager** during the lab — never hardcode it in your code.

- Sign up at the AI provider of your choice
- Generate an API key
- Save it somewhere safe — you will paste it into Secrets Manager in a later step

<!-- TODO: screenshot - nơi tạo API key trên provider -->
![AI API key](/images/5-Workshop/5.2-prerequisite/api-key.png)

#### Frontend assets

A pre-built static website is provided for this workshop. You do not need to write any frontend code.

- The site includes a sign-in page (Cognito Hosted UI), trip preference form, and itinerary result display
- Download and extract the zip file before Step 3

[Download frontend assets](#)

#### Tools

Install these tools before starting (if not already present):

```bash
# AWS CLI — https://aws.amazon.com/cli/
aws --version

# jq (optional, for JSON parsing in terminal)
jq --version
```

#### Estimated cost & time

- **Time:** approximately 2–3 hours to complete all steps
- **Cost:** minimal during development; production-scale costs depend on traffic volume. Use AWS Budgets alerts (Step 7) to monitor spending.
