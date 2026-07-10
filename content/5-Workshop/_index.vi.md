---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Wakan – Trợ lý du lịch cá nhân hóa bằng AI

#### Tổng quan

**Wakan** là ứng dụng web serverless gợi ý lịch trình du lịch tại Việt Nam. Thay vì để người dùng nhập prompt tự do, hệ thống cung cấp các lựa chọn có cấu trúc trên giao diện (đối tượng đi cùng, ngân sách, số ngày, loại trải nghiệm, phương tiện, đi gần/xa). Backend sẽ validate request, kiểm tra quota theo gói dịch vụ, ưu tiên trả kết quả từ cache nếu có, và chỉ gọi AI khi thật sự cần thiết thông qua một Lambda xử lý riêng.

Workshop này hướng dẫn bạn dựng kiến trúc AWS cốt lõi của Wakan:

- **Biên & frontend:** Amazon S3 + Amazon CloudFront (+ tùy chọn AWS WAF)
- **Xác thực & API:** Amazon Cognito + Amazon API Gateway
- **Xử lý nghiệp vụ:**  AWS Lambda AI Processor
- **Dữ liệu:** Amazon DynamoDB (Users/Subscriptions, Cache, Verified Places)
- **Bảo mật & kiểm soát chi phí:** AWS Secrets Manager, IAM least privilege, Amazon CloudWatch, AWS Budgets

Sau khi hoàn thành workshop, bạn sẽ có luồng end-to-end: đăng nhập → chọn tùy chọn chuyến đi → nhận lịch trình có cấu trúc, kèm kiểm soát quota và cache theo gói Free / Plus / Pro.

#### Nội dung

1. [Tổng quan workshop](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequiste/)
3. [Frontend & phân phối biên](5.3-Frontend-edge/)
4. [Xác thực & lớp API](5.4-Auth-API/)
5. [Lớp dữ liệu với DynamoDB](5.5-Data-layer/)
6. [Xử lý nghiệp vụ với Lambda](5.6-Business-logic/)
7. [Bảo mật, giám sát & tối ưu chi phí](5.7-Security-monitoring/)
8. [Dọn dẹp tài nguyên](5.8-Cleanup/)
