---
title: "Worklog Tuần 9"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:
* Hoàn thiện "não" của hệ thống: logic AI Processor — kiểm tra quota theo gói, tạo cache key chuẩn hóa, check cache trước khi gọi AI.
* Tham gia review chéo code với các team để đảm bảo logic quota/cache khớp với schema output AI.

### Các công việc đã hoàn thành:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Trạng thái |
| --- | --- | --- | --- | --- |
| 2 | - Review chéo code với team AI: kiểm tra prompt template và JSON schema output <br> - Góp ý luồng xử lý lỗi khi AI trả về định dạng sai | 15/06/2026 | 15/06/2026 | Hoàn thành |
| 3 | - Trả về 429 nếu vượt hạn mức kèm message gợi ý nâng cấp | 16/06/2026 | 16/06/2026 | Hoàn thành |
| 4 | - Xây dựng cache key: hash payload lựa chọn user + ngữ cảnh (thời tiết, khung giờ) <br> - Query RecommendationCache; nếu cache hit → trả kết quả, không invoke AI Processor | 17/06/2026 | 17/06/2026 | Hoàn thành |
| 5 | - Nếu cache miss → invoke Lambda AI Processor qua SDK InvokeCommand <br> - Nhận kết quả JSON, lưu vào bảng Cache, tăng usedToday/usedThisMonth +1 trong Users/Subscriptions | 18/06/2026 | 18/06/2026 | Hoàn thành |
| 6 | - Test end-to-end từng nhánh: quota exceeded, cache hit, cache miss + AI call <br> - Điều chỉnh error handling và log chi tiết từng bước | 19/06/2026 | 19/06/2026 | Hoàn thành |

### Kết quả đạt được:
* **Orchestrator hoàn chỉnh:** Đầy đủ 3 bước — check quota → check cache → invoke AI Processor — với error handling và logging đầy đủ.
* **Integration mượt mà:** Logic quota/cache khớp với schema output AI; cross-review với team AI và frontend giúp phát hiện sớm 2 lỗi mapping payload.