---
title: "Worklog Tuần 10"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:
* Debug và tối ưu luồng cache hit/miss, điều chỉnh TTL theo loại lịch trình (gần/xa/nhiều ngày).
* Tham gia test end-to-end lần đầu toàn hệ thống và họp fix bug liên phòng ban.

### Các công việc đã hoàn thành:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Trạng thái |
| --- | --- | --- | --- | --- |
| 2 | - Test end-to-end lần đầu: đăng nhập → chọn lựa chọn → nhận lịch trình <br> - Ghi nhận 3 lỗi: timeout Lambda, cache key không trùng, response format lệch | 22/06/2026 | 22/06/2026 | Hoàn thành |
| 3 | - Debug cache key: chuẩn hóa chuỗi hash, thêm fallback cho trường hợp payload thiếu tham số <br> - Tăng timeout Lambda AI Processor từ 3s lên 10s | 23/06/2026 | 23/06/2026 | Hoàn thành |
| 4 | - Tối ưu TTL theo loại lịch trình: ngắn ngày (1–2 ngày) → 6h, trung bình (3–5 ngày) → 12h, dài ngày (>5 ngày) → 24h <br> - Kiểm tra TTL hoạt động đúng qua DynamoDB console | 24/06/2026 | 24/06/2026 | Hoàn thành |
| 5 | - Sửa lỗi response format: chuẩn hóa JSON output Orchestrator theo API contract đã chốt tuần 8 <br> - Thêm validation layer trước khi trả về client | 25/06/2026 | 25/06/2026 | Hoàn thành |
| 6 | - Họp fix bug liên phòng ban: phân công lại task còn sót, cập nhật timeline  | 26/06/2026 | 26/06/2026 | Hoàn thành |

### Kết quả đạt được:
* **Cache hoạt động ổn định:** TTL phân theo độ dài chuyến giúp cân bằng giữa freshness và chi phí AI call — cache hit rate tăng từ ~30% lên ~65%.
* **E2E pass toàn bộ:** Sau 2 vòng debug, hệ thống chạy trơn tru cho cả 3 gói dịch vụ; không còn lỗi timeout hay format mismatch.