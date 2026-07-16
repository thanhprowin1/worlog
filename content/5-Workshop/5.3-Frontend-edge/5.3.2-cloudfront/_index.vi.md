---
title: "Tạo CloudFront distribution"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### Mục tiêu

Tạo CloudFront distribution để:

1. Phục vụ static assets từ S3 bucket (private, qua Origin Access Control)
2. Sau này chuyển tiếp request `/api/*` đến API Gateway (cấu hình ở Bước 5.4)

CloudFront trở thành **điểm truy cập duy nhất** của hệ thống Wakan.

#### Bước 1 — Tạo Origin Access Control (OAC)

1. Mở **CloudFront** → **Origin access** → **Create control setting**
2. Cấu hình:
   - **Name:** `wakan-oac`
   - **Signing behavior:** Sign requests (recommended)
   - **Origin type:** S3
3. Bấm **Create**

<!-- TODO: screenshot - form Create OAC -->


#### Bước 2 — Tạo distribution

1. Mở **CloudFront** → **Distributions** → **Create distribution**
2. **Origin** settings:
   - **Origin domain:** chọn S3 bucket `wakan-frontend-<your-unique-id>.s3.ap-southeast-1.amazonaws.com`
   - **Origin access:** Origin access control settings (recommended)
   - **Origin access control:** chọn `wakan-oac`
   - Để trống **Origin path**

<!-- TODO: screenshot - Origin settings khi tạo distribution -->
![Origin settings](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-origin.png)

3. **Default cache behavior:**
   - **Viewer protocol policy:** Redirect HTTP to HTTPS
   - **Allowed HTTP methods:** GET, HEAD, OPTIONS
   - **Cache policy:** CachingOptimized (hoặc CachingDisabled khi đang dev)
   - **Origin request policy:** CORS-S3Origin (nếu frontend cần CORS header từ origin)

4. **Settings:**
   - **Price class:** Prefer edge locations covering Asia (khuyến nghị cho người dùng Việt Nam: **PriceClass_200** hoặc All)
   - **Alternate domain name (CNAME):** để trống (dùng domain mặc định `*.cloudfront.net`)
   - **Default root object:** `index.html`
   - **WAF:** chưa bật (tùy chọn ở bước sau)

<!-- TODO: screenshot - Default cache behavior + Settings -->
![CloudFront settings](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-settings.png)

5. Bấm **Create distribution**

CloudFront sẽ hiện banner: **"The S3 bucket policy needs to be updated"**. Bấm **Copy policy** — bạn sẽ dán vào bucket ở bước tiếp theo.

<!-- TODO: screenshot - banner copy bucket policy sau khi create distribution -->
![Copy bucket policy banner](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-copy-policy.png)

#### Bước 3 — Cập nhật S3 bucket policy

1. Quay lại **S3** → bucket của bạn → **Permissions** → **Bucket policy** → **Edit**
2. Dán policy đã copy từ CloudFront. Nội dung tương tự:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::wakan-frontend-<your-unique-id>/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::<ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
                }
            }
        }
    ]
}
```

3. Bấm **Save changes**

<!-- TODO: screenshot - bucket policy đã dán và save -->


#### Bước 4 — Chờ deploy & kiểm tra

1. Quay lại **CloudFront** → distribution của bạn
2. Đợi **Status** chuyển sang **Enabled / Deployed** (thường 2–5 phút)

<!-- TODO: screenshot - distribution status Deployed -->
![Distribution deployed](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-deployed.png)

3. Copy **Distribution domain name** (ví dụ `dxxxxxxxxxxxx.cloudfront.net`)
4. Mở trên trình duyệt — bạn sẽ thấy frontend Wakan

<!-- TODO: screenshot - trình duyệt mở CloudFront domain hiện frontend Wakan -->
![Frontend qua CloudFront](/worlog/images/5-Workshop/5.3-Frontend-edge/frontend-live.png)

#### Ghi chú về `/api/*` (làm sau)

Ở Bước 5.4, sau khi API Gateway sẵn sàng, bạn sẽ thêm **origin thứ hai** (API Gateway) và một **cache behavior** cho path pattern `/api/*` để:

| Path | Origin | Cache |
|---|---|---|
| Default (`*`) | S3 bucket | Có cache |
| `/api/*` | API Gateway | Không cache (forward all) |

Bạn **chưa** cần cấu hình phần này bây giờ.

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Distribution status | **Deployed / Enabled** |
| OAC | Đã gắn vào S3 origin |
| Bucket policy | Chỉ cho phép CloudFront distribution này |
| HTTPS site | Mở được qua `https://dxxxx.cloudfront.net` |
| Truy cập S3 trực tiếp | Vẫn bị chặn (bucket private) |
