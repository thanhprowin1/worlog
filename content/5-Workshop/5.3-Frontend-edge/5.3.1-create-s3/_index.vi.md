---
title: "Tạo S3 bucket & upload frontend"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

#### Mục tiêu

Tạo một S3 bucket private và upload frontend đã build sẵn của Wakan. Bucket **không** public — người dùng chỉ truy cập site thông qua CloudFront (bước tiếp theo).

#### Bước 1 — Tạo bucket

1. Mở AWS Console → **S3** → **Create bucket**
2. Cấu hình:
   - **Bucket name:** `waka.id.vn`  
     *(Tên bucket S3 là unique toàn cầu — thêm initials hoặc hậu tố ngẫu nhiên)*
   - **AWS Region:** `ap-southeast-2 `
   - **Object Ownership:** ACLs disabled (recommended)
   - **Block Public Access:** giữ **tất cả** option bật (bucket vẫn private)
   - **Bucket Versioning:** Disabled
   - **Default encryption:** SSE-S3 (Amazon S3 managed keys)

<!-- TODO: screenshot - form Create bucket với các option trên -->
![Tạo S3 bucket](/worlog/images/5-Workshop/5.3-Frontend-edge/create-bucket.png)

3. Bấm **Create bucket**

<!-- TODO: screenshot - bucket đã tạo thành công trong danh sách -->
![Bucket đã tạo](/worlog/images/5-Workshop/5.3-Frontend-edge/TaoXongS3.png)

#### Bước 2 — Upload frontend assets

1. Mở bucket vừa tạo
2. Bấm **Upload**
3. Thêm toàn bộ file từ zip frontend đã giải nén (ở phần Prerequisites)
   - Cấu trúc điển hình: `index.html`, `assets/`, `css/`, `js/`, …
4. Bấm **Upload** và đợi trạng thái **Succeeded**

<!-- TODO: screenshot - danh sách file đã upload trong bucket -->
![File đã upload](/worlog/images/5-Workshop/5.3-Frontend-edge/uploaded-files.png)

Hoặc upload bằng AWS CLI:

```bash
aws s3 sync ./wakan-frontend/ s3://wakan-frontend-<your-unique-id>/ \
  --region ap-southeast-2
```

#### Bước 3 — Kiểm tra (tùy chọn)

```bash
# Liệt kê object trong bucket
aws s3 ls s3://wakan-frontend-<your-unique-id>/ --recursive --region ap-southeast-2
```

Bạn sẽ thấy `index.html` và các asset liên quan.

{{% notice note %}}
**Không bật Static website hosting** trên bucket này. Public website hosting không cần thiết — CloudFront sẽ phục vụ nội dung an toàn qua Origin Access Control (OAC) ở bước sau.
{{% /notice %}}

#### Checkpoint

| Mục | Kết quả mong đợi |
|---|---|
| Bucket tồn tại | Tên bắt đầu bằng `waka.id.vn` ở `ap-southeast-2` |
| Block Public Access | Cả 4 option đều **ON** |
| Object đã upload | Ít nhất có `index.html` |
| Public access | URL S3 trực tiếp **không** mở được website |
