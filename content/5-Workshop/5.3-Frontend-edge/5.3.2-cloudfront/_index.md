---
title: "Create CloudFront distribution"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### Objective

Create a CloudFront distribution that:

1. Serves static assets from the S3 bucket (private, via Origin Access Control)
2. Will later forward `/api/*` requests to API Gateway (configured in Step 5.4)

CloudFront becomes the **single entry point** of the Wakan system.

#### Step 1 — Create Origin Access Control (OAC)

1. Open **CloudFront** → **Origin access** → **Create control setting**
2. Configure:
   - **Name:** `wakan-oac`
   - **Signing behavior:** Sign requests (recommended)
   - **Origin type:** S3
3. Click **Create**

<!-- TODO: screenshot - form Create OAC -->


#### Step 2 — Create the distribution

1. Open **CloudFront** → **Distributions** → **Create distribution**
2. **Origin** settings:
   - **Origin domain:** select your S3 bucket `wakan-frontend-<your-unique-id>.s3.ap-southeast-1.amazonaws.com`
   - **Origin access:** Origin access control settings (recommended)
   - **Origin access control:** select `wakan-oac`
   - Leave **Origin path** empty

<!-- TODO: screenshot - Origin settings khi tạo distribution -->
![Origin settings](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-origin.png)

3. **Default cache behavior:**
   - **Viewer protocol policy:** Redirect HTTP to HTTPS
   - **Allowed HTTP methods:** GET, HEAD, OPTIONS
   - **Cache policy:** CachingOptimized (or CachingDisabled while developing)
   - **Origin request policy:** CORS-S3Origin (if your frontend needs CORS headers from origin)

4. **Settings:**
   - **Price class:** Use only North America and Europe *(or "Use all edge locations" if you want Asia coverage — recommended for Vietnam users: **PriceClass_200** or All)*
   - **Alternate domain name (CNAME):** leave empty for now (use the default `*.cloudfront.net` domain)
   - **Default root object:** `index.html`
   - **WAF:** Do not enable yet (optional in next sub-step)

<!-- TODO: screenshot - Default cache behavior + Settings -->
![CloudFront settings](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-settings.png)

5. Click **Create distribution**

CloudFront will show a banner: **"The S3 bucket policy needs to be updated"**. Click **Copy policy** — you will paste it into the bucket in the next step.

<!-- TODO: screenshot - banner copy bucket policy sau khi create distribution -->
![Copy bucket policy banner](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-copy-policy.png)

#### Step 3 — Update S3 bucket policy

1. Go back to **S3** → your bucket → **Permissions** → **Bucket policy** → **Edit**
2. Paste the policy copied from CloudFront. It should look similar to:

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

3. Click **Save changes**

<!-- TODO: screenshot - bucket policy đã dán và save -->


#### Step 4 — Wait for deployment & test

1. Back in **CloudFront** → your distribution
2. Wait until **Last modified / Status** shows **Enabled** / **Deployed** (usually 2–5 minutes)

<!-- TODO: screenshot - distribution status Deployed -->
![Distribution deployed](/worlog/images/5-Workshop/5.3-Frontend-edge/cf-deployed.png)

3. Copy the **Distribution domain name** (e.g. `dxxxxxxxxxxxx.cloudfront.net`)
4. Open it in a browser — you should see the Wakan frontend

<!-- TODO: screenshot - trình duyệt mở CloudFront domain hiện frontend Wakan -->
![Frontend via CloudFront](/worlog/images/5-Workshop/5.3-Frontend-edge/frontend-live.png)

#### Note about `/api/*` (for later)

In Step 5.4, after API Gateway is ready, you will add a **second origin** (API Gateway) and a **cache behavior** for path pattern `/api/*` so that:

| Path | Origin | Cache |
|---|---|---|
| Default (`*`) | S3 bucket | Cached |
| `/api/*` | API Gateway | Not cached (forward all) |

You do **not** need to configure this yet.

#### Checkpoint

| Item | Expected result |
|---|---|
| Distribution status | **Deployed / Enabled** |
| OAC | Attached to the S3 origin |
| Bucket policy | Allows only this CloudFront distribution |
| HTTPS site | Opens via `https://dxxxx.cloudfront.net` |
| Direct S3 access | Still blocked (private bucket) |
