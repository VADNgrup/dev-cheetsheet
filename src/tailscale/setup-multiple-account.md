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