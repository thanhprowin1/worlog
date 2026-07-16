---
title: "Sự kiện 1: Community Day"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

## Thông tin nhanh

| Hạng mục | Chi tiết |
|----------|----------|
| Tên | AWS First Cloud AI Journey Community Day |
| Thời gian | 09:00 · 23/05/2026 |
| Địa điểm | Tầng 26, Tháp Bitexco, 02 Hải Triều, TP.HCM |
| Vai trò của tôi | Người tham dự |
| Diễn giả chính | Tinh Truong, Anh Pham, Thinh Nguyen, Mai Nguyen, Uyen Le, Thao Nguyen, Duc Dao, Vy Lam |

---

## Tôi học được gì?

### CloudFront không chỉ là CDN
Buổi nói của **Thinh Nguyen** giúp tôi nhìn CloudFront như một **lớp bảo vệ ở biên**, không chỉ để tăng tốc tĩnh. Các ý đáng nhớ:
- **Flat-rate pricing** giảm lo “cháy bill” khi bị DDoS.
- **Origin Cloaking / VPC Origin** che hạ tầng gốc khỏi internet công cộng.
- Chặn request độc hại ngay edge trước khi vào origin.

→ Liên hệ sau này với Wakan: gắn WAF + CloudFront trước S3/API là lựa chọn hợp lý.

### Làm việc với LLM: context quan trọng hơn “model giỏi”
- **Tinh Truong:** output kém thường do **prompt/context** thiếu cấu trúc. Một khung tối thiểu: *mục tiêu – thông tin – ràng buộc – tiêu chí thành công*.
- **Duc Dao:** `temperature = 0` **không** đảm bảo kết quả giống hệt mỗi lần (sai số GPU, batching API). Thiết kế hệ thống nên **chịu được biến thiên** và cân nhắc `temp ≈ 0.1` thay vì tin tuyệt đối vào 0.

→ Gợi ý cho Lambda AI Processor: ép JSON schema + validate đầu ra, không tin raw text.

### Multi-agent và analytics “nói chuyện” với dữ liệu
- **Vy Lam:** case chấm tín dụng startup bằng **nhiều agent chuyên biệt** trên Amazon Bedrock, chạy trong VPC cô lập — thời gian xử lý giảm mạnh so với quy trình thủ công.
- **Anh Pham:** demo **QuickSight Q** — hỏi bằng ngôn ngữ tự nhiên để ra insight/báo cáo, giảm phụ thuộc viết SQL thủ công.

### 36 giờ prototype (LotusHacks)
Nhóm **Mai / Uyên / Thảo** kể về **UTMorpho**: sản phẩm bắt đầu từ nỗi đau thật khi chỉnh UI bằng AI. Kỹ thuật **smart-diffing** giúp tiết kiệm token khi AI sinh/sửa UI.

---

## Góc nhìn cá nhân

Community Day nối rõ hai nửa: **AI model** và **hạ tầng an toàn**. Với vai trò backend của Wakan, mình ghi nhận ba việc cần làm ngay từ đầu thiết kế:
1. Bảo vệ biên (CloudFront/WAF) trước khi mở API ra ngoài.
2. Prompt có khung + validate JSON, không phụ thuộc “may mắn” của model.
3. Ưu tiên chạy workload nhạy cảm trong môi trường cô lập (VPC) khi có thể.

### Ảnh tại sự kiện

![Community Day - ảnh 1](/worlog/images/Event1-1.png)
