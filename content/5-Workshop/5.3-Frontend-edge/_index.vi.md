---
title: "Frontend & phân phối biên"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Mục tiêu

Trong phần này, bạn sẽ triển khai frontend tĩnh của Wakan và đặt nó phía sau CDN toàn cầu.

Sau khi hoàn thành phần này, bạn sẽ có:

- Một S3 bucket private chứa static assets của website
- Một CloudFront distribution phục vụ site qua HTTPS
- (Tùy chọn) AWS WAF gắn vào CloudFront để lọc traffic độc hại

```
Trình duyệt người dùng
    │
    ▼
AWS WAF (tùy chọn)
    │
    ▼
Amazon CloudFront  ──► Amazon S3  (static assets: HTML / CSS / JS)
    │
    └──► /api/*  ──► Amazon API Gateway  (cấu hình ở Bước 5.4)
```

#### Nội dung

1. [Tạo S3 bucket & upload frontend](5.3.1-create-s3/)
2. [Tạo CloudFront distribution](5.3.2-cloudfront/)
3. [Gắn WAF (tùy chọn)](5.3.3-waf/)
