---
title: "Tạo Cognito User Pool"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### Mục tiêu

Tạo Cognito User Pool và App Client để người dùng Wakan đăng ký / đăng nhập và nhận JWT token dùng cho API Gateway.

#### Bước 1 — Tạo User Pool

1. Mở **Amazon Cognito** → **User pools** → **Create user pool**
2. **Sign-in experience**
   - **Cognito user pool sign-in options:** Email
   - (Tùy chọn) cho phép Username nếu muốn
3. **Security requirements**
   - Password policy: mặc định Cognito là đủ cho lab (hoặc min length 8)
   - Multi-factor authentication: **No MFA** (lab) / Optional cho production
4. **Sign-up experience**
   - Self-registration: **Enable**
   - Required attributes: **email**
   - Email verification: gửi email (mặc định Cognito)
5. **Message delivery**
   - Lab: **Send email with Cognito** (quota free giới hạn)
6. **Integrate your app**
   - **User pool name:** `User pool - weu7cv`
   - **App client name:** `Wakand_AI`
   - **Client secret:** **Không** tạo client secret (SPA / frontend tĩnh)
   - Authentication flows: bật `ALLOW_USER_SRP_AUTH`, `ALLOW_REFRESH_TOKEN_AUTH` (và Hosted UI nếu dùng)

<!-- TODO: screenshot - Create user pool / App client settings -->
![Tạo Cognito User Pool](/images/5-Workshop/5.4-Auth-API/cognito-create.png)

7. Review → **Create user pool**

<!-- TODO: screenshot - User pool overview sau khi tạo -->
![User pool đã tạo](/images/5-Workshop/5.4-Auth-API/cognito-created.png)

#### Bước 2 — Cấu hình Hosted UI (khuyến nghị)

Hosted UI cung cấp trang login có sẵn, không cần tự viết UI auth.

1. Mở user pool `User pool - weu7cv` → **App integration**
2. **Domain**
   - Tạo Cognito domain: `wakan-<your-unique-id>`  
     → URL Hosted UI dạng:  
     `https://wakan-<your-unique-id>.auth.ap-southeast-1.amazoncognito.com`
3. **App client** `Wakand_AI` → **Hosted UI** / **Login pages**
   - **Allowed callback URLs:**  
     `https://<your-cloudfront-domain>/`  
     `http://localhost:3000/` *(tùy chọn dev local)*
   - **Allowed sign-out URLs:** giống callback
   - **Identity providers:** Cognito user pool
   - **OAuth 2.0 grant types:** Authorization code grant
   - **OpenID Connect scopes:** `openid`, `email`, `profile`

<!-- TODO: screenshot - Hosted UI domain + callback URLs -->
![Cấu hình Hosted UI](/images/5-Workshop/5.4-Auth-API/cognito-hosted-ui.png)

4. Save changes

#### Bước 3 — Tạo test user

1. User pool → **Users** → **Create user**
2. Nhập email, đặt temporary password (hoặc đánh dấu email verified + permanent password cho lab)
3. Đăng nhập một lần qua Hosted UI và đổi mật khẩu nếu được yêu cầu

<!-- TODO: screenshot - user list với test user -->
![Test user](/images/5-Workshop/5.4-Auth-API/cognito-test-user.png)

#### Bước 4 — Lưu các identifier (dùng ở bước sau)

Ghi lại:

| Mục | Nơi lấy | Ví dụ |
|---|---|---|
| **User pool ID** | User pool overview | `ap-southeast-2_XXXXXXXXX` |
| **App client ID** | App integration → App clients | `1a2b3c4d5e6f...` |
| **Hosted UI domain** | App integration → Domain | `wakan-xxx.auth.ap-southeast-2.amazoncognito.com` |
| **Issuer URL** | `https://cognito-idp.ap-southeast-2.amazonaws.com/<USER_POOL_ID>` | dùng cho API Gateway authorizer |

{{% notice tip %}}
Cấu hình frontend (sau): domain Cognito, client ID, redirect URI = CloudFront URL. Sau khi login, app lưu **Access token** (hoặc ID token tùy authorizer) và gửi `Authorization: Bearer <token>` khi gọi API.
{{% /notice %}}

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| User pool | Tên `User pool - weu7cv` tồn tại |
| App client | `Wakand_AI`, **không** có client secret |
| Domain | Cognito domain đã active |
| Callback URL | Có CloudFront domain của bạn |
| Test user | Đăng nhập Hosted UI được và nhận token |
