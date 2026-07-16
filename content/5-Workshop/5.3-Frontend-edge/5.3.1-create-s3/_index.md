---
title: "Create S3 bucket & upload frontend"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

#### Objective

Create a private S3 bucket and upload the pre-built Wakan frontend. The bucket will **not** be publicly accessible — users will access the site only through CloudFront (next step).

#### Step 1 — Create the bucket

1. Open the AWS Console → **S3** → **Create bucket**
2. Configure:
   - **Bucket name:** `waka.id.vn`  
     *(S3 bucket names are globally unique — add your initials or a random suffix)*
   - **AWS Region:** `ap-southeast-2`
   - **Object Ownership:** ACLs disabled (recommended)
   - **Block Public Access:** keep **all** options enabled (bucket stays private)
   - **Bucket Versioning:** Disabled
   - **Default encryption:** SSE-S3 (Amazon S3 managed keys)

<!-- TODO: screenshot - form Create bucket với các option trên -->
![Create S3 bucket](/worlog/images/5-Workshop/5.3-Frontend-edge/create-bucket.png)

3. Click **Create bucket**

<!-- TODO: screenshot - bucket đã tạo thành công trong danh sách -->
![Bucket created](/worlog/images/5-Workshop/5.3-Frontend-edge/TaoXongS3.png)

#### Step 2 — Upload frontend assets

1. Open the bucket you just created
2. Click **Upload**
3. Add all files from the extracted frontend zip (from Prerequisites)
   - Typical structure: `index.html`, `assets/`, `css/`, `js/`, …
4. Click **Upload** and wait until the status is **Succeeded**

<!-- TODO: screenshot - danh sách file đã upload trong bucket -->
![Uploaded files](/worlog/images/5-Workshop/5.3-Frontend-edge/uploaded-files.png)

Alternatively, upload via AWS CLI:

```bash
aws s3 sync ./wakan-frontend/ s3://wakan-frontend-<your-unique-id>/ \
  --region ap-southeast-2
```

#### Step 3 — Verify (optional)

```bash
# List objects in the bucket
aws s3 ls s3://wakan-frontend-<your-unique-id>/ --recursive --region ap-southeast-2
```

You should see `index.html` and related assets.

{{% notice note %}}
**Do not enable Static website hosting** on this bucket. Public website hosting is not needed — CloudFront will serve the content securely through an Origin Access Control (OAC) in the next step.
{{% /notice %}}

#### Checkpoint

| Item | Expected result |
|---|---|
| Bucket exists | Name starts with `wakan-frontend-` in `ap-southeast-1` |
| Block Public Access | All 4 options **ON** |
| Objects uploaded | At least `index.html` is present |
| Public access | Direct S3 URL must **not** open the website |
