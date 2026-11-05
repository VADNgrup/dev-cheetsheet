# Hướng Dẫn Dọn Dẹp Và Nén Dung Lượng VHDX Ubuntu Trên WSL2

Tài liệu tóm tắt các bước tối ưu dung lượng đĩa ảo (`ext4.vhdx`) cho Ubuntu chạy trên WSL2.

---

## 🛠 Giai Đoạn 1: Dọn Dẹp Dung Lượng Trong Ubuntu

Mở terminal Ubuntu trên WSL2 và thực hiện các lệnh sau:

### 1. Xóa cache hệ thống & thư viện
```bash
# Xóa cache APT
sudo apt clean
sudo apt autoremove -y

# Xóa cache Conda / Pixi (nếu có)
conda clean --all -y
pixi clean

# Xóa log hệ thống cũ
sudo journalctl --vacuum-time=1d
```

### 2. Ghi đè ô nhớ trống bằng byte 0 (Để Windows nhận biết vùng đĩa trống)
```bash
sudo dd if=/dev/zero of=/zero.fill bs=1M status=progress
sudo rm /zero.fill
```
*Lưu ý: Lệnh `dd` sẽ chạy đến khi báo full đĩa ("No space left on device"), sau đó lệnh `rm` sẽ xóa file tạm này.*

---

## 🛠 Giai Đoạn 2: Nén (Compact) File VHDX Từ Windows

Sau khi dọn dẹp trong Ubuntu, chọn **một trong hai phương pháp** bên dưới để nén file trên Windows.

---

### Phương Pháp 1: Nén Trực Tiếp Bằng Lệnh `wsl --compact` (Nhanh & Khuyên Dùng)

Phương pháp này chỉ áp dụng cho Windows 11 / các phiên bản WSL2 mới.

1. Mở **PowerShell (Run as Administrator)**.
2. Dừng dịch vụ WSL để tránh bị khóa file:
   ```powershell
   wsl --shutdown
   Stop-Service wslservice -Force
   ```
3. Chạy lệnh nén trực tiếp tên distro của bạn:
   ```powershell
   wsl --compact Ubuntu-22.04
   ```
   *(Kiểm tra tên chính xác của distro bằng lệnh `wsl -l -v`)*

---

### Phương Pháp 2: Nén Bằng Công Cụ `Diskpart` (Truyền Thống)

Sử dụng phương pháp này nếu cách 1 không khả thi hoặc bạn dùng phiên bản Windows cũ hơn.

#### Bước 1: Ngắt hoàn toàn dịch vụ ngầm khóa file VHDX
Chạy trong **PowerShell (Run as Administrator)**:
```powershell
wsl --shutdown
Set-Service wslservice -StartupType Disabled
Stop-Service wslservice -Force
Get-Process *wsl* | Stop-Process -Force
```

#### Bước 2: Tiến hành nén bằng Diskpart
1. Mở `diskpart`:
   ```powershell
   diskpart
   ```
2. Thực hiện lần lượt các lệnh trong giao diện `DISKPART>` (thay đường dẫn tới file `.vhdx` của bạn):
   ```cmd
   select vdisk file="C:\Users\Duc-Anh\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_79rhkp1fndgsc\LocalState\ext4.vhdx"
   attach vdisk readonly
   compact vdisk
   detach vdisk
   exit
   ```

#### Bước 3: Bật lại dịch vụ WSL
Sau khi Diskpart báo nén thành công (`100%`), bật lại dịch vụ WSL trong **PowerShell**:
```powershell
Set-Service wslservice -StartupType Automatic
Start-Service wslservice
```

---

## 💡 Mẹo Tự Động Thu Nhỏ (Dành cho Windows 11)

Để WSL tự động giải phóng dung lượng thừa về cho Windows mỗi khi bạn xóa dữ liệu bên trong Ubuntu mà không cần chạy thủ công các bước trên:

Chạy lệnh sau trong PowerShell (Admin):
```powershell
wsl --manage Ubuntu-22.04 --set-sparse true
```
