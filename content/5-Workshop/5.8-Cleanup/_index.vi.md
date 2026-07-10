---
title: "Dọn dẹp tài nguyên"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Mục tiêu

Xóa toàn bộ tài nguyên workshop Wakan theo **thứ tự an toàn** để dừng chi phí và tránh lỗi phụ thuộc (ví dụ xóa bucket khi CloudFront vẫn đang dùng).

{{% notice warning %}}
Dọn dẹp **không hoàn tác** được. Hãy export screenshot, log hoặc dữ liệu mẫu cần cho báo cáo **trước** khi xóa.
{{% /notice %}}

#### Thứ tự xóa khuyến nghị

```
1. CloudFront  (disable → xóa distribution; gỡ association WAF)
2. WAF         (Web ACL)
3. API Gateway (API + usage plans + API keys)
4. Lambda      ( AI Processor)
5. Cognito     (User pool + domain)
6. DynamoDB    (các bảng)
7. Secrets Manager
8. S3          (xóa hết object → xóa bucket)
9. IAM roles & CloudWatch (alarms, dashboard, log groups)
10. AWS Budgets (tùy chọn)
```

#### Nội dung

1. [Xóa tài nguyên biên & API](5.8.1-edge-api/)
2. [Xóa compute, dữ liệu & secrets](5.8.2-compute-data/)
3. [Xóa IAM, giám sát & kiểm tra cuối](5.8.3-iam-monitor/)
