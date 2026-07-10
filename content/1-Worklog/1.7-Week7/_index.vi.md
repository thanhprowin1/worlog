---
title: "Worklog Tuần 7"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:
* Chốt phạm vi dự án Wakan, phác thảo luồng nghiệp vụ và schema dữ liệu cho phần backend.
* Tham gia dựng môi trường làm việc chung: Git, IAM user cá nhân, quy tắc branching.

### Các công việc đã hoàn thành:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Trạng thái |
| --- | --- | --- | --- | --- |
| 2 | - Tham gia tạo Git repo, quy tắc nhánh (main/dev/release) | 01/06/2026 | 01/06/2026 | Hoàn thành |
| 3 | - Viết use-case chi tiết cho luồng "tìm chuyến đi": danh sách lựa chọn, ràng buộc, edge case <br> - Phác thảo sơ bộ schema DynamoDB (Users, Cache, PremiumLocations) | 02/06/2026 | 02/06/2026 | Hoàn thành |
| 4 | - Rà soát sơ đồ kiến trúc do Anh Đức vẽ, góp ý luồng AI Processor <br> - Ghi nhận các điểm cần đồng bộ API contract giữa các team | 03/06/2026 | 03/06/2026 | Hoàn thành |
| 5 | - Tạo IAM user riêng cho nhóm, đề xuất policy tối thiểu cần thiết <br> - Setup branch protection rule trên repo | 04/06/2026 | 04/06/2026 | Hoàn thành |
| 6 | - Tổng hợp biên bản họp, gửi lại toàn nhóm <br> - Chốt timeline nội bộ cho 5 tuần còn lại | 05/06/2026 | 05/06/2026 | Hoàn thành |

### Kết quả đạt được:
* **Use-case & schema:** Tài liệu use-case "tìm chuyến đi" đã liệt kê đầy đủ tham số đầu vào (loại trải nghiệm, ngân sách, đối tượng đi cùng…) và schema DynamoDB sơ bộ làm nền tảng cho tuần sau.
* **Quy trình nhóm:** Git repo hoạt động, branch protection rule đã bật, mỗi thành viên có IAM user riêng — không còn lo xung đột code hay quyền hạn.