---
title: "Worklog Tuần 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:
* Load test nhẹ, tối ưu cấu hình Lambda (memory/timeout) và review toàn bộ luồng nghiệp vụ lần cuối.
* Tham gia họp tổng duyệt kết quả kiểm thử trước khi chốt bản demo.

### Các công việc đã hoàn thành:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Trạng thái |
| --- | --- | --- | --- | --- |
| 2 | - Họp tổng duyệt kết quả kiểm thử toàn nhóm <br> - Ghi nhận các điểm cần tối ưu: latency AI call, cold start Lambda, memory usage | 29/06/2026 | 29/06/2026 | Hoàn thành |
| 3 | - Load test nhẹ: 20 request đồng thời qua API Gateway, đo p50/p95 latency <br> - Phát hiện cold start AI Processor ~1.2s; AI Processor ~2.5s | 30/06/2026 | 30/06/2026 | Hoàn thành |
| 4 | - Tăng memory Lambda AI Processor từ 128MB lên 256MB (giảm duration ~30%) <br> - Tăng memory AI Processor từ 256MB lên 512MB; điều chỉnh timeout 15s | 01/07/2026 | 01/07/2026 | Hoàn thành |
| 5 | - Kiểm tra edge case: user chưa có record Users → auto-init; cache key collision | 02/07/2026 | 02/07/2026 | Hoàn thành |
| 6 | - Vá 2 bug còn sót: race condition khi 2 request cùng lúc tăng usedToday <br> - Dùng DynamoDB UpdateItem atomic counter thay vì read-modify-write | 03/07/2026 | 03/07/2026 | Hoàn thành |

### Kết quả đạt được:
* **Hiệu năng ổn định:** Sau tối ưu memory/timeout, p95 latency giảm từ ~4.5s xuống ~2.8s; cold start không còn là bottleneck chính.
* **Logic nghiệp vụ cứng:** Edge case được cover, race condition được vá bằng atomic counter — sẵn sàng cho demo.