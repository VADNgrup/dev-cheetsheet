# 📖 Dev Cheatsheet & Dictionary Web

Hệ thống tra cứu tài liệu & cheatsheet nội bộ dành cho dev team.

---

## 🚀 Cách chạy Web tra cứu Local

Mở terminal tại thư mục dự án và chạy:

`ash
# Cách 1: Chạy file script có sẵn
chmod +x start.sh
./start.sh

# Hoặc cách 2: Chạy trực tiếp bằng Python
python3 -m http.server 3000
`

Sau đó mở trình duyệt truy cập: **[http://localhost:3000](http://localhost:3000)**

---

## 🌟 Tính năng
- ⚡ **Zero-build:** Không cần compile/build, viết Markdown là web cập nhật ngay khi refresh.
- 🔍 **Tìm kiếm toàn văn (Full-text search):** Tìm nhanh lệnh, cú pháp hoặc từ khóa bất kỳ.
- 📋 **One-click Copy Code:** Nút copy code trên mọi code block.
- 🎨 **Giao diện hiện đại & thanh lịch:** Tối ưu hiển thị code, bảng biểu và danh mục.

---

## 🗂 Cấu trúc tài liệu

`	ext
dev-cheetsheet/
├── index.html           # Cấu hình giao diện Docsify
├── _sidebar.md          # Menu danh mục hiển thị bên trái
├── start.sh             # Script khởi động server nhanh
└── src/                 # Chứa toàn bộ nội dung markdown
    ├── common_bashcommand/
    │   ├── ssh.md
    │   ├── sync-data.md
    │   └── user-management.md
    └── tailscale/
        └── setup-multiple-account.md
`

---

## ✍️ Thêm tài liệu mới vào Dictionary

1. Tạo file .md mới trong thư mục tương ứng ở src/ (hoặc tạo thư mục mới nếu là công nghệ mới, ví dụ: src/docker/cheatsheet.md).
2. Mở file _sidebar.md và thêm liên kết:
   `markdown
   - **Docker**
     - [Cheatsheet](src/docker/cheatsheet.md)
   `
3. Lưu file và reload lại web trên trình duyệt!