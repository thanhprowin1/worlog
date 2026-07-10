---
title: "Frontend & Edge Distribution"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Goal

In this section, you will deploy the Wakan static frontend and put it behind a global CDN.

By the end of this section, you will have:

- A private S3 bucket hosting the static website assets
- A CloudFront distribution serving the site over HTTPS
- (Optional) AWS WAF attached to CloudFront to filter malicious traffic

```
User browser
    │
    ▼
AWS WAF (optional)
    │
    ▼
Amazon CloudFront  ──► Amazon S3  (static assets: HTML / CSS / JS)
    │
    └──► /api/*  ──► Amazon API Gateway  (configured in Step 5.4)
```

#### Content

1. [Create S3 bucket & upload frontend](5.3.1-create-s3/)
2. [Create CloudFront distribution](5.3.2-cloudfront/)
3. [Attach WAF (optional)](5.3.3-waf/)
