title: Quản Lý Người Dùng, Nhóm Và Phân Quyền Trong Hệ Điều Hành Linux
tags:
linux
sysadmin
user-management
permissions
chmod
chown
sudoers
keywords:
useradd
usermod
groupadd
chmod
chown
chgrp
umask
sudoers
visudo
passwd
file permissions
rwx
sticky bit
suid
sgid
date: 2026-11-04

# Quản Lý Người Dùng, Nhóm Và Phân Quyền Trong Linux
Tài liệu hướng dẫn chi tiết các lệnh và phương pháp quản lý tài khoản người dùng (users), nhóm (groups), phân quyền tập tin/thư mục (file permissions) và cấp quyền quản trị (sudo) trên hệ điều hành Linux.
---
1. Quản Lý Người Dùng (User Management)
Tạo Người Dùng Mới
```bash
# Tạo user mới với thư mục home (/home/username) và shell mặc định bash
sudo useradd -m -s /bin/bash <username>

# Đặt mật khẩu cho user vừa tạo (hoặc đổi mật khẩu user hiện tại)
sudo passwd <username>
```
Chỉnh Sửa & Xóa Người Dùng
```bash
# Đổi tên người dùng (Username)
sudo usermod -l <new_username> <old_username>

# Khóa (lock) tài khoản user
sudo usermod -L <username>

# Mở khóa (unlock) tài khoản user
sudo usermod -U <username>

# Xóa user (giữ lại thư mục home)
sudo userdel <username>

# Xóa user cùng toàn bộ thư mục home và mailbox
sudo userdel -r <username>
```
---
2. Quản Lý Nhóm (Group Management)
Tạo & Xóa Nhóm
```bash
# Tạo nhóm mới
sudo groupadd <groupname>

# Xóa nhóm
sudo groupdel <groupname>
```
Quản Lý Thành Viên Trong Nhóm
```bash
# Thêm user vào một nhóm (giữ nguyên các nhóm cũ của user)
sudo usermod -aG <groupname> <username>

# Xem danh sách các nhóm mà user thuộc về
groups <username>

# Xóa user khỏi một nhóm cụ thể
sudo gpasswd -d <username> <groupname>
```
---
3. Cấp Quyền Sudo (Quản Trị Viên)
Cách 1: Thêm user vào nhóm sudo/wheel
Trên Ubuntu/Debian:
```bash
  sudo usermod -aG sudo <username>
  ```
Trên RHEL/CentOS/Rocky Linux:
```bash
  sudo usermod -aG wheel <username>
  ```
Cách 2: Chỉnh sửa trực tiếp file sudoers bằng `visudo`
```bash
sudo visudo
```
Thêm dòng sau vào cuối file để cấp quyền sudo không cần gõ mật khẩu (tùy chọn):
```text
<username> ALL=(ALL:ALL) NOPASSWD: ALL
```
---
4. Quản Lý Phân Quyền Tập Tin & Thư Mục (File Permissions)
Cấu trúc phân quyền Linux gồm 3 nhóm: User (u), Group (g), Others (o) với 3 quyền cơ bản:
r (Read - 4): Quyền đọc file / xem danh sách file trong thư mục.
w (Write - 2): Quyền ghi, chỉnh sửa file / tạo, xóa file trong thư mục.
x (Execute - 1): Quyền thực thi file script / truy cập (`cd`) vào thư mục.
Thay Đổi Quyền Bằng Numeric Mode (Số)
```bash
# Phân quyền 755 (u=rwx, g=rx, o=rx) cho thư mục
chmod 755 /path/to/directory

# Phân quyền 644 (u=rw, g=r, o=r) cho tập tin
chmod 644 /path/to/file.txt

# Phân quyền 700 (chỉ owner có toàn quyền) cho thư mục bảo mật (VD: ~/.ssh)
chmod 700 ~/.ssh

# Áp dụng đệ quy cho toàn bộ cây thư mục và file bên trong
chmod -R 755 /path/to/directory
```
Thay Đổi Quyền Bằng Symbolic Mode (Ký tự)
```bash
# Thêm quyền thực thi (x) cho chủ sở hữu (u)
chmod u+x script.sh

# Bỏ quyền ghi (w) của group (g) và others (o)
chmod go-w document.txt

# Gán quyền đọc/ghi cho tất cả mọi người (a = all)
chmod a+rw shared.txt
```
---
5. Thay Đổi Chủ Sở Hữu (Ownership)
```bash
# Đổi Owner của file/thư mục
sudo chown <username> /path/to/file

# Đổi cả Owner và Group cùng lúc
sudo chown <username>:<groupname> /path/to/file

# Đổi Owner/Group cho thư mục và tất cả tập tin bên trong (Recursive)
sudo chown -R ducanh:nvidia /home/ducanh/data
```
---
6. Phân Quyền Nâng Cao (Special Permissions)
SUID (4000): Cho phép người chạy file thực thi với quyền của Owner file đó.
```bash
  chmod u+s /path/to/binary
  ```
SGID (2000): Mọi file tạo ra trong thư mục sẽ tự động kế thừa Group sở hữu của thư mục đó.
```bash
  chmod g+s /path/to/shared_folder
  ```
Sticky Bit (1000): Chỉ owner của file mới có quyền xóa file đó trong thư mục chung (Ví dụ: thư mục `/tmp`).
```bash
  chmod +t /path/to/shared_folder
  ```
---
7. Bảng Tổng Hợp Kiểm Tra Nhanh (Cheat Sheet)
Lệnh	Ý nghĩa / Mức độ sử dụng
`id <user>`	Xem UID, GID và các group của user
`whoami`	Kiểm tra tài khoản hiện tại đang đăng nhập
`chmod 600 id_rsa`	Đặt quyền chuẩn cho Private Key SSH
`chmod 700 ~/.ssh`	Đặt quyền chuẩn cho thư mục SSH
`ls -la`	Xem danh sách file bao gồm phân quyền và owner
