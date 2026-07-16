---
title: "Bản đề xuất"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Wakan - Trợ Lý Du Lịch Cá Nhân Hóa Bằng AI
## Giải pháp AWS Serverless hợp nhất cho việc gợi ý lịch trình du lịch cá nhân hóa và tin cậy

### 1. Tóm tắt điều hành
**Wakan** là hệ thống trợ lý du lịch cá nhân hóa bằng AI tại Việt Nam, giúp người dùng tự động thiết kế lịch trình tối ưu dựa trên vị trí địa lý thực tế, ngân sách, thời gian rảnh, phương tiện di chuyển, đối tượng đi cùng và loại trải nghiệm mong muốn. 

Bằng cách áp dụng kiến trúc hoàn toàn Serverless trên AWS (bao gồm AWS WAF, Amazon CloudFront, Amazon S3, Amazon API Gateway, Amazon Cognito, AWS Lambda, Amazon DynamoDB, AWS Secrets Manager và Amazon CloudWatch), dự án hướng tới việc giảm thiểu chi phí vận hành, nâng cao tính bảo mật, và đảm bảo khả năng tự động mở rộng linh hoạt theo số lượng người dùng.

---

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại
Việc lên lịch trình du lịch thủ công là một quy trình tốn nhiều thời gian và công sức. Người dùng phải tự tìm hiểu thông tin rời rạc từ nhiều nguồn (mạng xã hội, blog, bản đồ) rồi tự chắp vá lịch trình, đường đi và ngân sách. 

Khi sử dụng các mô hình AI dạng chat tự do (như ChatGPT hay Gemini), người dùng gặp hai rào cản lớn:
1. **Độ phức tạp khi viết Prompt**: Người dùng phải biết cách viết prompt chi tiết (Prompt Engineering) mới nhận được kết quả có cấu trúc mong muốn.
2. **AI Ảo tưởng (Hallucination)**: Mô hình AI thường tự bịa ra các địa điểm không có thật, đưa ra khoảng cách địa lý sai lệch hoặc gợi ý lộ trình không phù hợp với thực tế giao thông tại Việt Nam.

#### Giải pháp
**Wakan** giải quyết triệt để các vấn đề này bằng cách thiết kế giao diện web dạng lựa chọn có sẵn thay vì cho phép người dùng nhập prompt tự do. Người dùng chỉ cần tích chọn các thông số (ví dụ: đi một mình/gia đình, ngân sách thấp/cao, đi gần/xa, đi trong ngày/nhiều ngày).
- **Kiểm soát đầu vào (Input Control)**: Giới hạn lựa chọn trên UI giúp kiểm soát dữ liệu đầu vào tốt hơn, loại bỏ rủi ro tấn công *Prompt Injection*.
- **Kiểm tra ở Backend**: Dữ liệu gửi lên vẫn được kiểm tra chặt chẽ ở Lambda Orchestrator để tránh request bất hợp lệ trực tiếp qua API.
- **Chuẩn hóa đầu ra (Structured Outputs)**: Ép mô hình AI trả về dữ liệu tuân thủ strict JSON schema cố định để frontend dễ dàng hiển thị.
- **Xác thực địa điểm (Verified Places)**: Tích hợp cơ sở dữ liệu các địa điểm đã kiểm chứng cho gói Pro để loại bỏ hoàn toàn tình trạng AI tự sinh thông tin sai lệch.

#### Lợi ích và Hoàn vốn đầu tư (ROI)
- **Tiết kiệm thời gian**: Giúp người dùng tiết kiệm đến 90% thời gian lên kế hoạch du lịch nhờ tạo lịch trình tự động chỉ trong vài giây.
- **Tối ưu chi phí Serverless**: Không mất chi phí duy trì máy chủ rảnh. Chi phí phát sinh theo lượng sử dụng thực tế và hoàn toàn nằm trong gói AWS Credits của nhóm.
- **Tối ưu chi phí gọi AI**: Sử dụng cơ chế cache thông minh trên DynamoDB dựa trên tọa độ vị trí làm tròn và các lựa chọn của người dùng, giúp trả kết quả lập tức cho các yêu cầu tương tự mà không cần gọi lại API AI đắt đỏ.

---

### 3. Kiến trúc giải pháp
Hệ thống được thiết kế theo mô hình Serverless trên AWS nhằm đảm bảo tính bảo mật và khả năng co giãn tự động.

![Kiến trúc hệ thống Wakan](/worlog/images/2-Proposal/kientruc.jpg)

#### Dịch vụ AWS sử dụng
- **AWS WAF**: Lọc các request bất thường, chống bot và các cuộc tấn công tầng ứng dụng (HTTP/HTTPS).
- **Amazon CloudFront**: Phân phối giao diện web tĩnh từ S3 tới người dùng toàn cầu với độ trễ thấp và chuyển tiếp request API `/api/*` tới API Gateway.
- **Amazon S3**: Lưu trữ an toàn các asset tĩnh của Frontend (HTML, CSS, JS).
- **Amazon API Gateway**: Cổng REST API chính, tích hợp xác thực token và quản lý Usage Plans để giới hạn tần suất request.
- **Amazon Cognito**: Quản lý đăng ký, đăng nhập và xác thực phiên làm việc bằng JSON Web Token (JWT).
- **AWS Lambda**: Được chia thành hai hàm xử lý độc lập để tách biệt trách nhiệm:
  1. **AWS Lambda (Orchestrator)**: Điều phối nghiệp vụ chính: kiểm tra gói người dùng và quota sử dụng từ DynamoDB, tính toán cache key và lưu lịch trình mới.
  2. **AWS Lambda (AI Processor)**: Giao tiếp trực tiếp với External AI API, lấy API Key bảo mật từ Secrets Manager, tạo prompt có cấu trúc và validate JSON đầu ra.
- **Amazon DynamoDB**: Cơ sở dữ liệu NoSQL lưu trữ 3 bảng chính:
  1. `Users/Subscriptions`: Lưu thông tin tài khoản, gói dịch vụ (Free/Plus/Pro) và quota lượt dùng còn lại trong ngày.
  2. `Cache`: Lưu trữ các hành trình đã tạo kèm theo cơ chế tự hủy TTL (6-12 giờ cho chuyến đi ngắn, 24-48 giờ cho chuyến đi dài).
  3. `Verified Places`: Danh sách địa điểm đã được nhóm kiểm chứng thực tế nhằm phục vụ cho gói Pro.
- **AWS Secrets Manager**: Lưu trữ an toàn các API Key của bên thứ ba (AI API).
- **Amazon CloudWatch**: Ghi nhận logs, theo dõi metrics và phát cảnh báo (CloudWatch Alarms) khi hệ thống gặp lỗi.

#### Thiết kế thành phần
1. **Phân phối và Biên dịch**: WAF và CloudFront bảo vệ biên hệ thống. CloudFront tự động định tuyến traffic: file tĩnh tải từ S3, REST API chuyển tiếp tới API Gateway.
2. **Xác thực người dùng**: Đăng ký/đăng nhập qua Cognito. API Gateway tích hợp Cognito Authorizer để kiểm tra JWT hợp lệ của request trước khi chuyển tiếp vào backend xử lý.
3. **Logic nghiệp vụ & AI**:
   - **Lambda Orchestrator** kiểm tra quota người dùng. Tiếp đó, làm tròn tọa độ người dùng để tạo cache key đồng bộ, truy vấn bảng `Cache`. Nếu có cache, trả kết quả ngay. Nếu không, gọi Lambda AI Processor.
   - **Lambda AI Processor** lấy API key từ Secrets Manager, dựng prompt từ dữ liệu đầu vào đã kiểm tra, gọi mô hình AI và kiểm tra định dạng đầu ra trước khi trả về cho Orchestrator để ghi cache và hiển thị cho người dùng.
4. **Các gói sử dụng dịch vụ**:
   - **Free**: Trải nghiệm cơ bản, giới hạn số lượt gợi ý trong ngày.
   - **Plus**: Hạn mức cao hơn, mở rộng gợi ý hành trình dài ngày và di chuyển xa.
   - **Pro**: Trải nghiệm cao cấp, AI chỉ được sử dụng dữ liệu từ bảng `Verified Places` đã kiểm chứng để loại bỏ hoàn toàn thông tin ảo tưởng.

---

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai
Dự án được triển khai trong vòng 6 tuần (từ Tuần 7 đến Tuần 12) theo lộ trình thực tập:
- **Giai đoạn 1: Thiết kế và Lập kế hoạch (Tuần 7)**: Viết use-case chi tiết, xác định cấu trúc bảng DynamoDB và thiết kế wireframe trên Google Drawings/Jamboard cho giao diện.
- **Giai đoạn 2: Thiết lập hạ tầng AWS (Tuần 8)**: Khởi tạo S3, CloudFront, Cognito User Pool và phân quyền IAM theo nguyên tắc đặc quyền tối thiểu.
- **Giai đoạn 3: Hiện thực hóa Logic Backend & Dữ liệu (Tuần 9)**: Phát triển Lambda Orchestrator (quota, cache), thiết lập Secrets Manager, API Gateway Usage Plans và chuẩn bị bộ dữ liệu địa điểm kiểm chứng.
- **Giai đoạn 4: Tích hợp Frontend & API (Tuần 10)**: Hoàn thiện giao diện UI động, tích hợp bản đồ, kết nối API đăng nhập và gọi API gợi ý lịch trình.
- **Giai đoạn 5: Kiểm thử và Tối ưu hóa (Tuần 11)**: Thiết lập CloudWatch Logs/Alarms, chạy thử nghiệm tải (load test), tối ưu cold-start Lambda và pentest kiểm tra bảo mật (WAF, IAM).
- **Giai đoạn 6: Hoàn thiện và Trình diễn (Tuần 12)**: Kiểm thử đầu cuối lần cuối, quay video demo sản phẩm, viết báo cáo tổng kết và chuẩn bị thuyết trình.

#### Yêu cầu kỹ thuật
- **Hạ tầng dưới dạng mã nguồn (IaC)**: Định nghĩa tài nguyên AWS thông qua CDK hoặc Serverless Framework để dễ tái sử dụng.
- **Ngôn ngữ Backend**: Node.js/TypeScript chạy trên Lambda giúp tối ưu tốc độ và giảm thời gian khởi động (cold start).
- **Định dạng dữ liệu AI**: Sử dụng các mô hình hỗ trợ đầu ra dạng Structured Outputs (JSON Schema).
- **Dịch vụ Bản đồ**: Kết hợp Google Maps API hoặc Amazon Location Service để tính khoảng cách và vẽ tuyến đường.

---

### 5. Lộ trình & Mốc triển khai

| Tuần | Thành viên | Công việc dự kiến |
|---|---|---|
| **Tuần 7** | **Cả nhóm** | Họp kickoff, thống nhất phạm vi 3 gói (Free/Plus/Pro), chốt sơ đồ kiến trúc và tạo Git repo chung, cấp tài khoản IAM. |
| | Trần Hữu Khang | Viết use-case chi tiết luồng nghiệp vụ & phác thảo DynamoDB schema. |
| | Lê Trần Anh Đức | Hoàn thiện tài liệu kiến trúc hệ thống và dự thảo chính sách bảo mật IAM. |
| | Đinh Hoàng Vương | Vẽ wireframe trên Google Drawings/Jamboard cho giao diện lựa chọn lịch trình. |
| | Trần Phúc Đăng | Nghiên cứu các External AI API và nháp prompt template ban đầu. |
| **Tuần 8** | **Cả nhóm** | Thống nhất API contract (request/response) giữa Frontend – Backend – AI. |
| | Trần Hữu Khang | Dựng khung API Gateway và khung Lambda Orchestrator. |
| | Lê Trần Anh Đức | Cấu hình WAF, CloudFront, S3 hosting frontend và Cognito User Pool. |
| | Đinh Hoàng Vương | Dựng giao diện tĩnh các bước lựa chọn lịch trình, tích hợp Login Cognito. |
| | Trần Phúc Đăng | Thiết lập Lambda AI Processor và test gọi LLM qua SDK. |
| **Tuần 9** | **Cả nhóm** | Đánh giá chéo mã nguồn để đảm bảo sự đồng bộ về dữ liệu đầu ra/đầu vào. |
| | Trần Hữu Khang | Hoàn thiện logic nghiệp vụ Orchestrator: kiểm tra quota gói, băm cache key tọa độ. |
| | Lê Trần Anh Đức | Cấu hình Secrets Manager (AI Key) và API Gateway Usage Plans. |
| | Đinh Hoàng Vương | Kết nối giao diện với API thật, xử lý hiệu ứng tải (loading) và hiển thị lỗi. |
| | Trần Phúc Đăng | Tối ưu prompt template, xây dựng bảng dữ liệu địa điểm `Verified Places` trên DynamoDB. |
| **Tuần 10** | **Cả nhóm** | Chạy thử nghiệm tích hợp đầu cuối (end-to-end) từ UI đến kết quả AI và sửa lỗi. |
| | Trần Hữu Khang | Tối ưu hóa luồng cache hit/miss và tinh chỉnh thời gian TTL theo loại hành trình. |
| | Lê Trần Anh Đức | Dựng bảng theo dõi CloudWatch Logs/Alarms và cấu hình AWS Budgets cảnh báo chi phí. |
| | Đinh Hoàng Vương | Vẽ lịch trình chi tiết lên bản đồ trực quan, tối ưu hóa responsive di động. |
| | Trần Phúc Đăng | Kiểm chứng chất lượng gợi ý từ AI đối với cả 3 gói dịch vụ. |
| **Tuần 11** | **Cả nhóm** | Họp tổng kết kết quả kiểm thử, rà soát các điểm thắt nút cổ chai (bottlenecks). |
| | Trần Hữu Khang | Chạy load test, điều chỉnh tài nguyên cấu hình Lambda (Memory/Timeout). |
| | Lê Trần Anh Đức | Thực hiện pentest cơ bản (WAF bypass, IAM auditing). |
| | Đinh Hoàng Vương | Tinh chỉnh giao diện hoàn thiện, chuẩn bị kịch bản demo ứng dụng. |
| | Trần Phúc Đăng | Viết test case toàn diện, chạy kiểm thử unit + integration. |
| **Tuần 12** | **Cả nhóm** | Chạy thử toàn bộ hệ thống, quay video demo, hoàn thiện tài liệu thuyết trình và nộp bài. |

---

### 6. Ước tính ngân sách
Với tính chất Serverless, chi phí vận hành hệ thống được tối ưu ở mức cực thấp:
- **AWS Lambda**: 0.00 USD (Miễn phí 1 triệu request mỗi tháng thuộc AWS Free Tier).
- **Amazon DynamoDB**: ~0.50 USD/tháng (chạy theo chế độ On-Demand linh hoạt).
- **Amazon S3 & CloudFront**: ~0.20 USD/tháng cho lưu trữ và truyền tải giao diện web.
- **Amazon Cognito**: 0.00 USD (Miễn phí cho 50,000 người dùng hoạt động hàng tháng - MAU).
- **AWS Secrets Manager**: 0.40 USD/tháng (Lưu trữ 1 API Key bí mật cho AI).
- **AWS WAF**: ~6.00 USD/tháng (Cước nền Web ACL 5 USD và 1 USD cho các quy tắc bảo mật).
- **Amazon CloudWatch**: ~0.50 USD/tháng cho logs cơ bản và alarms.
- **External AI API**: Trả tiền theo lượng token tiêu thụ thực tế (sử dụng credit miễn phí ban đầu).

**Tổng chi phí hạ tầng ước tính**: **~7.60 USD / tháng** (chưa bao gồm chi phí token AI), hoàn toàn nằm trong khoản AWS Credit tài trợ của đội nhóm.

---

### 7. Đánh giá rủi ro

#### Ma Trận Rủi Ro
- **AI Ảo tưởng / Sai dữ liệu**: Ảnh hưởng cao, Xác suất trung bình.
- **Vượt chi phí API AI**: Ảnh hưởng trung bình, Xác suất cao.
- **Spam request phá hoại API**: Ảnh hưởng trung bình, Xác suất trung bình.
- **Rò rỉ API Key nhạy cảm**: Ảnh hưởng nghiêm trọng, Xác suất thấp.

#### Chiến lược giảm thiểu
- **Hạn chế ảo tưởng**: Ép AI sử dụng dữ liệu từ bảng `Verified Places` đối với gói Pro. Ràng buộc prompt chỉ cho phép sắp xếp và chọn lọc thay vì sáng tạo tự do.
- **Kiểm soát chi phí AI**: Triển khai cơ chế cache thông minh dựa trên tọa độ vị trí làm tròn. Lưu kết quả cache lên tới 48 tiếng cho chuyến đi dài ngày.
- **Chống spam**: Bắt buộc xác thực qua Cognito, áp dụng Usage Plans trên API Gateway để rate limit và giới hạn quota cứng tại Lambda Orchestrator.
- **Bảo vệ khóa bí mật**: Lưu trữ tập trung tại Secrets Manager. Phân quyền IAM truy cập chặt chẽ, tuyệt đối không hardcode API Key vào mã nguồn git.

---

### 8. Kết quả kỳ vọng
1. **Ứng dụng MVP hoàn chỉnh**: Giao diện trực quan, cho phép người dùng lựa chọn sở thích du lịch và nhận lịch trình tối ưu, chính xác.
2. **Hạ tầng AWS Serverless tối ưu**: Hệ thống tự động co giãn theo số lượng người dùng, chi phí duy trì tiệm cận 0 USD khi không hoạt động.
3. **Bộ tài liệu kỹ thuật chuẩn chỉ**: Báo cáo thiết kế API, cấu hình bảo mật, kiểm thử và phân tích chi phí vận hành thực tế.
