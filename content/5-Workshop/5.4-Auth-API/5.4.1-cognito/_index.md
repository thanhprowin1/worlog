---
title: "Create Cognito User Pool"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### Objective

Create a Cognito User Pool and App Client so Wakan users can sign up / sign in and receive JWT tokens used by API Gateway.

#### Step 1 — Create User Pool

1. Open **Amazon Cognito** → **User pools** → **Create user pool**
2. **Sign-in experience**
   - **Cognito user pool sign-in options:** Email
   - (Optional) also allow Username if you prefer
3. **Security requirements**
   - Password policy: Cognito defaults are fine for the lab (or set min length 8)
   - Multi-factor authentication: **No MFA** (lab) / Optional for production
4. **Sign-up experience**
   - Self-registration: **Enable**
   - Required attributes: **email**
   - Email verification: send email message (Cognito default)
5. **Message delivery**
   - For the lab: **Send email with Cognito** (limited free quota)
6. **Integrate your app**
   - **User pool name:** `User pool - weu7cv`
   - **App client name:** `Wakand_AI`
   - **Client secret:** Do **not** generate a client secret (public SPA / static frontend)
   - Authentication flows: allow `ALLOW_USER_SRP_AUTH`, `ALLOW_REFRESH_TOKEN_AUTH` (and Hosted UI if enabled)

<!-- TODO: screenshot - Create user pool / App client settings -->
![Create Cognito User Pool](/worlog/images/5-Workshop/5.4-Auth-API/cognito-create.png)

7. Review → **Create user pool**

<!-- TODO: screenshot - User pool overview sau khi tạo -->
![User pool created](/worlog/images/5-Workshop/5.4-Auth-API/cognito-created.png)

#### Step 2 — Configure Hosted UI (recommended)

Hosted UI gives you a ready-made login page without writing auth UI.

1. Open user pool `User pool - weu7cv` → **App integration**
2. **Domain**
   - Create a Cognito domain: `wakan-<your-unique-id>`  
     → Hosted UI URL will look like:  
     `https://wakan-<your-unique-id>.auth.ap-southeast-2.amazoncognito.com`
3. **App client** `Wakand_AI` → **Hosted UI** / **Login pages**
   - **Allowed callback URLs:**  
     `https://<your-cloudfront-domain>/`  
     `http://localhost:3000/` *(optional for local dev)*
   - **Allowed sign-out URLs:** same as above
   - **Identity providers:** Cognito user pool
   - **OAuth 2.0 grant types:** Authorization code grant
   - **OpenID Connect scopes:** `openid`, `email`, `profile`

<!-- TODO: screenshot - Hosted UI domain + callback URLs -->
![Hosted UI settings](/worlog/images/5-Workshop/5.4-Auth-API/cognito-hosted-ui.png)

4. Save changes

#### Step 3 — Create a test user

1. User pool → **Users** → **Create user**
2. Enter email, set a temporary password (or mark email as verified and set permanent password for lab speed)
3. Sign in once via Hosted UI and change password if required

<!-- TODO: screenshot - user list với test user -->
![Test user](/worlog/images/5-Workshop/5.4-Auth-API/cognito-test-user.png)

#### Step 4 — Save identifiers (you will need them later)

Write down:

| Item | Where to find | Example |
|---|---|---|
| **User pool ID** | User pool overview | `ap-southeast-2_XXXXXXXXX` |
| **App client ID** | App integration → App clients | `1a2b3c4d5e6f...` |
| **Hosted UI domain** | App integration → Domain | `wakan-xxx.auth.ap-southeast-1.amazoncognito.com` |
| **Issuer URL** | `https://cognito-idp.ap-southeast-2.amazonaws.com/<USER_POOL_ID>` | used by API Gateway authorizer |

{{% notice tip %}}
Frontend config (later): set Cognito domain, client ID, redirect URI = your CloudFront URL. After login, the app stores the **Access token** (or ID token, depending on your authorizer setup) and sends it as `Authorization: Bearer <token>` on API calls.
{{% /notice %}}

#### Checkpoint

| Item | Expected result |
|---|---|
| User pool | Name `User pool - weu7cv` exists |
| App client | `Wakand_AI`, **no** client secret |
| Domain | Cognito domain active |
| Callback URL | Includes your CloudFront domain |
| Test user | Can sign in via Hosted UI and receive tokens |
