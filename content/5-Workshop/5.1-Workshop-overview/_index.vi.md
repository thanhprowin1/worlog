---
title: "Tổng quan workshop"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Bạn sẽ xây dựng gì

Trong workshop này, bạn sẽ triển khai kiến trúc serverless cốt lõi của **Wakan** — một ứng dụng gợi ý lịch trình du lịch tại Việt Nam nhờ AI. Sau khi hoàn thành, bạn sẽ có luồng end-to-end hoạt động:

> Người dùng mở website → đăng nhập qua Cognito → chọn tùy chọn chuyến đi trên giao diện → API Gateway chuyển hướng đến Lambda Orchestrator → Orchestrator kiểm tra cache và quota → gọi Lambda AI Processor → trả về lịch trình có cấu trúc.

#### Kiến trúc tổng quan

<!-- TODO: screenshot/diagram - kiến trúc tổng quan Wakan -->
![Kiến trúc Wakan](/images/5-Workshop/5.1-Workshop-overview/architecture.jpg)

Hệ thống được chia thành 4 lớp:

**Lớp 1 — Biên & Frontend**
- Amazon S3 lưu trữ ứng dụng web tĩnh (HTML/CSS/JS).
- Amazon CloudFront phân phối nội dung toàn cầu và chuyển tiếp các request `/api/*` đến API Gateway.
- AWS WAF (tùy chọn) lọc các truy cập HTTP/HTTPS độc hại ở biên.

**Lớp 2 — Xác thực & API**
- Amazon Cognito quản lý đăng ký, đăng nhập và token JWT cho người dùng.
- Cognito Authorizer trên API Gateway xác thực mọi request trước khi chuyển tiếp.
- Usage Plan áp dụng giới hạn tốc độ theo từng gói dịch vụ.

**Lớp 3 — Xử lý nghiệp vụ (Lambda)**
- **Lambda Orchestrator:** validate đầu vào, kiểm tra quota người dùng từ DynamoDB, tạo cache key từ tọa độ làm tròn + tùy chọn, trả kết quả cache nếu có, nếu không thì chuyển sang AI Processor.
- **Lambda AI Processor:** lấy API key từ Secrets Manager, xây dựng prompt có cấu trúc, gọi mô hình AI bên ngoài, ép định dạng JSON đầu ra và xử lý lỗi fallback.

**Lớp 4 — Dữ liệu (DynamoDB)**
- **Users/Subscriptions:** thông tin người dùng, gói hiện tại (Free/Plus/Pro), số lượt dùng còn lại mỗi ngày.
- **Cache:** lịch trình đã tạo trước đó, key dựa trên tọa độ làm tròn + tùy chọn; TTL thay đổi theo độ dài chuyến (6–12h cho ngắn ngày, 24–48h cho nhiều ngày).
- **Verified Places:** dữ liệu địa điểm đã kiểm chứng, chỉ dùng cho gói Pro để ngăn AI "ảo tưởng".

#### Luồng request đi như thế nào

<!-- TODO: screenshot/diagram - sơ đồ sequence flow request -->


1. Người dùng truy cập `www.wakan.id.vn` — CloudFront phục vụ static assets từ S3.
2. Người dùng đăng nhập qua Cognito Hosted UI → nhận ID token + access token.
3. Người dùng chọn tùy chọn chuyến đi trên UI (đối tượng đi cùng, ngân sách, số ngày, phương tiện, loại trải nghiệm).
4. Frontend gửi POST `/api/itinerary` kèm token trong header Authorization.
5. API Gateway xác thực JWT qua Cognito Authorizer → chuyển tiếp đến Lambda Orchestrator.
6. Orchestrator đọc gói & quota người dùng từ DynamoDB → tạo cache key.
7. Nếu cache hit → trả ngay lịch trình (không gọi AI).
8. Nếu cache miss → invoke Lambda AI Processor.
9. AI Processor lấy API key từ Secrets Manager → xây prompt → gọi AI model → validate JSON output → lưu kết quả vào bảng Cache → trả về Orchestrator.
10. Orchestrator trừ quota trong DynamoDB → trả lịch trình về frontend.


#### Những phần không nằm trong phạm vi workshop

Workshop này tập trung vào core serverless backend. Các phần sau được lược bỏ có chủ đích:
- Phát triển frontend code (sử dụng sẵn static site)
- Tích hợp thời tiết / bản đồ
- Thiết lập Amazon Location Service
- Pipeline CI/CD
