---
title: Đồng Bộ Và Di Chuyển Dữ Liệu Trong Linux Bằng Rsync Và SCP
tags:
  - linux
  - sysadmin
  - rsync
  - scp
  - data-migration
  - backup
  - ssh
keywords:
  - rsync
  - scp
  - rsync resume
  - rsync progress
  - rsync delete
  - remote copy
  - sync files
  - migrate data
  - bandwidth limit
  - ssh copy
date: 2026-11-04
---

# Đồng Bộ Và Di Chuyển Dữ Liệu Nâng Cao Bằng Rsync Và SCP

Tài liệu hướng dẫn chi tiết cách sử dụng `rsync` và `scp` để di chuyển dữ liệu lớn, backup, và đồng bộ tập tin giữa các máy chủ cục bộ hoặc qua kết nối mạng an toàn (SSH).

---

## 1. So Sánh Nhanh Rsync Và SCP

| Tiêu chí | Rsync | SCP (Secure Copy) |
| :--- | :--- | :--- |
| **Cơ chế** | Chỉ truyền các phần dữ liệu bị thay đổi (Delta transfer). | Copy toàn bộ file bất kể đã tồn tại hay chưa. |
| **Hỗ trợ Resume** | **Có** (rất thích hợp cho file/dataset lớn). | **Không** (bị ngắt giữa chừng phải copy lại từ đầu). |
| **Giữ Thuộc Tính File** | Giữ nguyên permissions, owner, timestamps, symlinks. | Giới hạn (cần cờ `-p`). |
| **Tính Năng Lọc** | Hỗ trợ exclude/include theo mẫu file (`--exclude`). | Không hỗ trợ lọc nâng cao. |
| **Ứng Dụng Khuyên Dùng**| Đồng bộ dataset, backup định kỳ, truyền file lớn qua mạng. | Copy nhanh 1-2 file đơn lẻ giữa các máy. |

---

## 2. Sử Dụng Rsync Nâng Cao (Khuyên Dùng)

Cú pháp chuẩn: `rsync [OPTIONS] SOURCE DESTINATION`

### Các Tham Số Thường Dùng Nhất
* `-a` (*archive*): Chế độ lưu trữ, giữ nguyên permissions, owner, timestamps, symlinks.
* `-v` (*verbose*): Hiển thị chi tiết danh sách file đang truyền.
* `-z` (*compress*): Nén dữ liệu trong quá trình truyền giúp tiết kiệm băng thông.
* `-P` (*progress & partial*): Hiển thị thanh tiến trình thực thời và cho phép **tiếp tục (resume)** nếu bị rớt mạng.
* `-h` (*human-readable*): Hiển thị dung lượng dạng Dễ đọc (MB, GB).

---

### Các Kịch Bản Sử Dụng Rsync Thực Tế

#### 1. Đồng bộ dữ liệu local lên Máy chủ từ xa (Spark/DGX)
```bash
rsync -avzP /path/to/local/data/ user@remote_host:/path/to/remote/data/
```

#### 2. Đồng bộ dữ liệu qua Cổng SSH tùy chỉnh (ví dụ Port 2222)
```bash
rsync -avzP -e 'ssh -p 2222' /local/data/ user@remote_host:/remote/data/
```

#### 3. Tạo bản sao chính xác (Mirror) – Xóa file ở đích nếu nguồn đã xóa
```bash
rsync -avzP --delete /local/data/ user@remote_host:/remote/data/
```

#### 4. Loại bỏ các file/thư mục không muốn truyền (`--exclude`)
```bash
rsync -avzP --exclude='.git' --exclude='*.tmp' --exclude='node_modules' \
      /project/ user@remote_host:/project/
```

#### 5. Giới hạn băng thông truyền dữ liệu (tránh nghẽn mạng)
```bash
# Giới hạn tốc độ tối đa 50MB/s (50000 KBytes/sec)
rsync -avzP --bwlimit=50000 /huge_dataset/ user@remote_host:/destination/
```

---

<Callout title="Cảnh Báo Về Dấu Gạch Chéo (/) Ở Đường Dẫn Nguồn" type="warning">
* <code>rsync -avz /data/src/ /dst/</code> (Có dấu <code>/</code> ở cuối): Copy <b>nội dung bên trong</b> thư mục <code>src</code> vào <code>dst</code>.
* <code>rsync -avz /data/src /dst/</code> (Không có dấu <code>/</code>): Copy <b>cả bản thân thư mục <code>src</code></b> vào bên trong <code>dst</code>.
</Callout>

---

## 3. Sử Dụng SCP (Secure Copy Protocol)

Cú pháp chuẩn: `scp [OPTIONS] SOURCE DESTINATION`

### Các Lệnh SCP Cơ Bản

#### 1. Gửi file đơn lẻ từ máy local lên remote
```bash
scp /path/to/file.txt user@remote_host:/path/to/destination/
```

#### 2. Gửi nguyên thư mục (Recursive) từ local lên remote
```bash
scp -r /path/to/directory user@remote_host:/path/to/destination/
```

#### 3. Tải file từ máy remote về máy local
```bash
scp user@remote_host:/path/to/remote_file.txt /local/destination/
```

#### 4. Sử dụng cổng SSH tùy chỉnh và nén dữ liệu
```bash
scp -P 2222 -C -r /local/dir user@remote_host:/remote/dir
```

---

## 4. Bảng Tra Cứu Nhanh Lệnh Rsync (Quick Reference)

```bash
# Kịch bản khuyên dùng cho AI/ML Dataset Migration lớn (Resume được + Nén + Báo tiến trình):
rsync -avzP -e "ssh -o ServerAliveInterval=15" /dataset/ user@spark-ip:/data/dataset/
```