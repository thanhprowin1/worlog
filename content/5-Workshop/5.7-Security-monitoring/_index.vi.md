---
title: "Bảo mật, giám sát & tối ưu chi phí"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Mục tiêu

Trong phần này bạn sẽ tăng cường bảo mật cho Wakan, thiết lập giám sát bằng CloudWatch và cấu hình giới hạn chi phí để việc gọi AI không vượt kiểm soát.

```
Bảo mật                          Giám sát                        Kiểm soát chi phí
─────────                         ───────────                   ──────────────
• IAM least privilege             • CloudWatch Logs             • Cảnh báo AWS Budgets
• Secrets Manager                 • Custom metrics              • Cache → giảm gọi AI
• Tọa độ làm tròn (riêng tư)      • API Gateway metrics         • Quota theo gói
• WAF ở biên                      • Lambda metrics              • IAM deny xóa resource
```

{{% notice note %}}
Bước này là **quan sát và tăng cường** — không tạo resource mới làm thay đổi luồng chức năng. Bạn đang thêm hàng rào xung quanh những gì đã dựng ở Bước 5.1–5.6.
{{% /notice %}}

#### Nội dung

1. [Rà soát IAM least privilege](5.7.1-iam-review/)
2. [Giám sát CloudWatch](5.7.2-cloudwatch/)
3. [Cảnh báo AWS Budgets](5.7.3-budgets/)
