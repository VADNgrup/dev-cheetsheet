---
title: Các Tùy Chọn Cấu Hình Và Lệnh SSH Thường Dùng Trên Linux
tags:
  - linux
  - ssh
  - sysadmin
  - networking
  - security
  - ssh-config
keywords:
  - ssh
  - ssh key
  - ssh config
  - ssh tunneling
  - port forwarding
  - ssh-keygen
  - ssh-copy-id
  - ssh agent
  - proxycommand
  - jump host
date: 2026-11-04
---

# Hướng Dẫn Các Tùy Chọn Cấu Hình Và Lệnh SSH Nâng Cao

Tài liệu tóm tắt các tùy chọn dòng lệnh SSH (CLI options), cách quản lý SSH Keys, kỹ thuật Port Forwarding/Tunneling và cách tối ưu file cấu hình `~/.ssh/config`.

---

## 1. Cú Pháp Và Tùy Chọn CLI Thông Dụng

Cú pháp cơ bản: `ssh [options] user@hostname`

### Các Tham Số (Flags) Quan Trọng

| Option | Tác dụng | Ví dụ |
| :--- | :--- | :--- |
| `-i <identity_file>` | Chỉ định file Private Key dùng để đăng nhập | `ssh -i ~/.ssh/id_ed25519 user@host` |
| `-p <port>` | Chỉ định cổng kết nối SSH (mặc định là 22) | `ssh -p 2222 user@host` |
| `-N` | Không mở interactive shell (dùng khi forward port) | `ssh -N -L 8080:localhost:80 user@host` |
| `-f` | Đưa tiến trình SSH xuống chạy ngầm (background) | `ssh -f -N -L ...` |
| `-C` | Bật chế độ nén dữ liệu luồng kết nối | `ssh -C user@host` |
| `-v` / `-vvv` | Chế độ Verbose (In log chi tiết, dùng debug lỗi) | `ssh -vvv user@host` |
| `-X` / `-Y` | Bật tính năng X11 Forwarding (chạy ứng dụng GUI) | `ssh -X user@host` |
| `-o <Option>` | Truyền trực tiếp option cấu hình SSH | `ssh -o StrictHostKeyChecking=no user@host` |

---

## 2. Quản Lý SSH Keys (Xác Thực Không Mật Khẩu)

### Bước 1: Khởi tạo cặp SSH Key (Khuyên dùng ED25519)
```bash
ssh-keygen -t ed25519 -C "ducanh@spark-server"
```

### Bước 2: Copy Public Key lên Máy chủ từ xa
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 22 user@remote_host
```
*(Nếu cổng SSH đổi thành 2222: `ssh-copy-id -i ~/.ssh/id_ed25519.pub "-p 2222" user@remote_host`)*

---

## 3. SSH Tunneling & Port Forwarding (Chuyển Hướng Cổng)

### 1. Local Port Forwarding (`-L`)
Cho phép truy cập dịch vụ trên máy remote (ví dụ: Jupyter Notebook chạy port 8888 trên máy Spark) từ trình duyệt local:
```bash
# Mở cổng 8888 trên máy local trỏ về cổng 8888 của máy remote
ssh -N -L 8888:localhost:8888 user@remote_host
```
*Sau khi chạy, truy cập trình duyệt local tại: `http://localhost:8888`*

### 2. Remote Port Forwarding (`-R`)
Chia sẻ dịch vụ đang chạy ở máy local cho máy remote truy cập:
```bash
ssh -N -R 9000:localhost:3000 user@remote_host
```

### 3. Dynamic Port Forwarding (`-D` - SOCKS Proxy)
Biến máy remote thành SOCKS Proxy server để duyệt web qua IP của máy đó:
```bash
ssh -N -D 1080 user@remote_host
```

---

## 4. Quản Lý File Cấu Hình SSH Client (`~/.ssh/config`)

Thay vì phải gõ câu lệnh SSH dài dòng kèm theo IP, port và key path, bạn tạo file `~/.ssh/config` để rút gọn thao tác.

### Ví Dụ File Cấu Hình Chuẩn (`~/.ssh/config`):

```text
# Cấu hình mặc định cho tất cả các Host
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_ed25519

# Cấu hình riêng cho máy NVIDIA DGX Spark
Host spark
    HostName 100.x.y.z
    User ducanh
    Port 22
    IdentityFile ~/.ssh/spark_key

# Cấu hình đi qua máy trung gian (Jump Host / Bastion)
Host spark-internal
    HostName 192.168.1.50
    User ducanh
    ProxyJump user@bastion-server.com
```

### Phân Tích Các Tùy Chọn Tối Ưu Trong Config:
* `ServerAliveInterval 60`: Tự gửi packet giữ kết nối mỗi 60 giây, **tránh bị dis/timeout SSH**.
* `ServerAliveCountMax 3`: Ngắt kết nối nếu gửi 3 lần packet kiểm tra mà máy chủ không phản hồi.
* `ProxyJump`: Kết nối tới máy chủ nội bộ thông qua một máy Bastion công cộng.

### Sử Dụng Sau Khi Cấu Hình Config:
Bây giờ bạn chỉ cần gõ lệnh ngắn gọn:
```bash
ssh spark
```
*(Hệ thống sẽ tự động dùng đúng User, IP, Port và SSH Key đã định nghĩa).*