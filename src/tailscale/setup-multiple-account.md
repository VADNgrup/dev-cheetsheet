title: Quản lý và Chạy nhiều Tài khoản Tailscale trên cùng một Máy Linux / NVIDIA DGX Spark
tags:
  - tailscale
  - linux
  - multi-account
  - networking
  - systemd
  - sysadmin
  - nvidia-spark
keywords:
  - tailscale
  - tailscaled
  - multi instance tailscale
  - tailscale switch
  - tailscale socket
  - tailscale tun
  - tailscale daemon
  - tailnet
  - create tailscale socket
  - manage tailscale accounts
  - run multiple tailscale
  - linux networking
  - rsync spark
  - reset linux password
date: 2026-11-04
---

# Hướng Dẫn Khởi Tạo Và Quản Lý Nhiều Tài Khoản Tailscale Trên Cùng Một Máy Linux

Tài liệu này hướng dẫn cách cấu hình, chạy song song và quản lý nhiều instance/tài khoản Tailscale (Tailnet) độc lập trên cùng một máy chủ Linux (đã thử nghiệm thành công trên NVIDIA DGX Spark / Ubuntu).

---

## 1. Nguyên Lý Chạy Nhiều Instance Tailscale Song Song

Mặc định, `tailscaled` sẽ khởi tạo interface `tailscale0` và dùng file socket mặc định `/run/tailscale/tailscaled.sock`. Khi muốn chạy song song instance thứ 2, 3... (ví dụ tài khoản `tista`, `account2`...), **bắt buộc mỗi instance phải sở hữu 3 thông số riêng biệt**:

1. `--socket`: Đường dẫn file socket giao tiếp riêng.
2. `--statedir`: Thư mục chứa file cấu hình và dữ liệu phiên (`tailscaled.state`) riêng.
3. `--port`: Cổng UDP riêng (tránh xung đột với port mặc định `41641`).
4. `--tun`: Tên interface mạng virtual riêng (ví dụ: `tailscale1`, `tailscale-tista`), tránh đụng độ với `tailscale0`.

---

## 2. Cấu Trúc Thư Mục Quản Lý Tập Trung

Khuyên dùng cấu trúc lưu dữ liệu và socket trong thư mục người dùng để dễ backup và quản lý:

```text
/home/<username>/.tailscale/
├── run/
│   ├── tailscaled-tista.sock
│   ├── tailscaled-acc2.sock
│   └── ...
└── data/
    ├── tista/
    ├── acc2/
    └── ...

```

Khởi tạo nhanh cấu trúc thư mục bằng lệnh:

```bash
mkdir -p ~/.tailscale/run
mkdir -p ~/.tailscale/data/tista
mkdir -p ~/.tailscale/data/acc2

```

---

## 3. Khởi Chạy Và Đăng Nhập Thủ Công (CLI)

### Bước 1: Khởi chạy Daemon cho tài khoản phụ (`tista`)

```bash
sudo tailscaled --socket=/home/ducanh/.tailscale/run/tailscaled-tista.sock \
                --statedir=/home/ducanh/.tailscale/data/tista \
                --port=40001 \
                --tun=tailscale-tista &

```

* **Xác nhận (Verification):** Chạy `ls -la ~/.tailscale/run/` để đảm bảo file `tailscaled-tista.sock` đã được tạo.

### Bước 2: Đăng nhập và Authenticate

Trỏ lệnh `tailscale` đến đúng socket vừa tạo:

```bash
sudo tailscale --socket=/home/ducanh/.tailscale/run/tailscaled-tista.sock up

```

* **Thực hiện:** Mở liên kết URL in ra trên terminal bằng trình duyệt và xác nhận cấp phép tài khoản.
* **Xác nhận (Verification):** Kiểm tra trạng thái mạng của account mới:
```bash
sudo tailscale --socket=/home/ducanh/.tailscale/run/tailscaled-tista.sock status

```



---

## 4. Tối Ưu Lệnh Điều Khiển Bằng Alias (`~/.bashrc`)

Để không phải gõ lại đường dẫn `--socket` dài dòng, thêm các dòng sau vào file `~/.bashrc`:

```bash
# Tailscale Multi-account Aliases
alias ts-main="sudo tailscale"
alias ts-tista="sudo tailscale --socket=/home/ducanh/.tailscale/run/tailscaled-tista.sock"
alias ts-acc2="sudo tailscale --socket=/home/ducanh/.tailscale/run/tailscaled-acc2.sock"

```

Nạp lại cấu hình:

```bash
source ~/.bashrc

```

**Cách dùng sau khi tạo alias:**

* Kiểm tra status: `ts-tista status`
* Ngắt kết nối: `ts-tista down`
* Kết nối lại: `ts-tista up`

---

## 5. Tự Động Khởi Chạy Cùng Hệ Thống Với Systemd

Để các tài khoản phụ tự chạy lại sau khi reboot máy Spark, tạo service template systemd:

1. **Tạo file service:**
```bash
sudo nano /etc/systemd/system/tailscale-multi@.service

```


2. **Dán cấu hình:**
```ini
[Unit]
Description=Tailscale instance for %i
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/sbin/tailscaled --socket=/home/ducanh/.tailscale/run/tailscaled-%i.sock --statedir=/home/ducanh/.tailscale/data/%i --port=40001 --tun=tailscale-%i
Restart=on-failure

[Install]
WantedBy=multi-user.target

```


3. **Kích hoạt service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tailscale-multi@tista

```



* **Xác nhận (Verification):** Chạy `sudo systemctl status tailscale-multi@tista` để thấy trạng thái `active (running)`.

---

với nhiều user, nên tách thành nhiều file và chỉ rõ đường dẫn, port sẽ ngon hơn

## 6. Giải Quyết Xử Lý Lỗi Thường Gặp (Troubleshooting)

### Lỗi 1: `tstun.New("tailscale0"): device or resource busy`

* **Nguyên nhân:** Quên khai báo tham số `--tun`, làm cho daemon phụ cố tình dùng chung `tailscale0` với daemon chính.
* **Khắc phục:** Thêm tham số `--tun=<tên_interface_mới>` (ví dụ: `--tun=tailscale1` hoặc `--tun=tailscale-tista`).

### Lỗi 2: `TPM: error opening: stat /dev/tpmrm0: no such file or directory`

* **Nguyên nhân:** Máy không có chip phần cứng TPM hoặc driver TPM chưa nạp.
* **Khắc phục:** Đây là cảnh báo (*warning*), không làm gián đoạn hoạt động của Tailscale. Bỏ qua cảnh báo này nếu thấy dòng `Engine created.` tiếp theo.

### Lỗi 3: `timeout khi ssh `
có thể kiểm tra port
sudo ufw allow 40001/udp
sudo ufw allow 40002/udp
---

## 7. Phụ Lục Lệnh Liên Quan (Cheat Sheet)

| Thao tác | Lệnh thực hiện |
| --- | --- |
| **Đếm số node trong Tailnet** | `tailscale status | grep -v "offline" | wc -l` |
| **Chuyển profile (Single Instance)** | `sudo tailscale switch <nickname>` |
| **Reset mật khẩu Linux** | `sudo passwd <username>` |
| **Rsync dữ liệu sang Spark** | `rsync -avzP /path/source/ user@spark_ip:/path/dest/` |

```

