---
trigger: always_on
description: - **Bắt buộc:** Luôn trả lời, đề xuất, và sinh code comment bằng **tiếng Việt**.
---

# HƯỚNG DẪN CHO GITHUB COPILOT - TOÀN BỘ REPOSITORY

## Ngôn ngữ trả lời mặc định
- **Bắt buộc:** Luôn trả lời, đề xuất, và sinh code comment bằng **tiếng Việt**.

---

## 1. Bối cảnh Dự án và Nguồn tri thức
Dự án này là một nền tảng **cho thuê phòng trọ theo mô hình managed marketplace**.

**NGUỒN TRI THỨC CHÍNH:** Toàn bộ đặc tả nghiệp vụ chi tiết, các tác nhân và luồng sự kiện được ghi lại trong file **`docs/use-cases-v1.2.md`**. Hãy luôn ưu tiên tham chiếu file này khi được hỏi về logic nghiệp vụ.

### Tóm tắt các Tác nhân chính:
* **Khách hàng (Customer):** Người dùng cuối có nhu cầu tìm kiếm và thuê một nơi ở phù hợp.
* **Chủ dự án (Project Owner):** Cá nhân hoặc tổ chức sở hữu bất động sản cho thuê.
* **Nhân viên Bán hàng (Sales Staff):** Đại diện tuyến đầu của công ty, chịu trách nhiệm trực tiếp trong việc hỗ trợ và dẫn dắt khách hàng.
* **Nhân viên Điều hành (Operator):** Quản trị viên nội bộ, chịu trách nhiệm duy trì tính toàn vẹn và chất lượng của nền tảng.
* **Quản trị viên Hệ thống (System Administrator):** Người dùng kỹ thuật cấp cao nhất, chịu trách nhiệm về sức khỏe, an ninh và cấu hình tổng thể của nền tảng.

---

## 2. Stack Công nghệ
-   **Backend:** Node.js (Express.js, JavaScript) - `server/`
-   **Frontend:** React (Vite, JavaScript/JSX) - `client/`
-   **Database:** MySQL/MariaDB
-   **Icons:** React Icons (Heroicons v2) - `react-icons@5.4.0`

### Ghi chú cho giai đoạn Development
- Backend dev lắng nghe cổng `5000` (xem `server/index.js`). Nếu dùng reverse proxy (Nginx), cấu hình `proxy_pass http://backend:5000/`.
- Kết nối DB dùng `mysql2` (không ORM). Đọc cấu hình từ biến môi trường: `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE`.
- Redis/Workers/MinIO là tùy chọn trong dev; bắt buộc khi lên staging/production.

### Cấu trúc Frontend (chuẩn hóa)
```
client/src/
├── pages/
│   ├── ChuDuAn/              # Module Chủ dự án (DARK LUXURY THEME)
│   │   ├── Dashboard.jsx/.css         # Tổng quan với metrics cards
│   │   ├── QuanLyTinDang.jsx/.css     # Danh sách tin đăng (table view)
│   │   ├── QuanLyTinDang_new.jsx/.css # Room display logic thông minh
│   │   ├── TaoTinDang.jsx/.css        # Form tạo tin (validation + upload)
│   │   ├── BaoCaoHieuSuat.jsx/.css    # Báo cáo với time filters
│   │   └── index.js                   # Export tất cả components
│   ├── login/                # Trang đăng nhập
│   ├── dangky/               # Trang đăng ký
│   └── trangchu/             # Trang chủ công khai
├── components/
│   ├── ChuDuAn/
│   │   ├── NavigationChuDuAn.jsx/.css # Sidebar navigation collapsible
│   │   ├── ModalTaoNhanhDuAn.jsx      # Modal tạo dự án nhanh
│   │   ├── ModalCapNhatDuAn.jsx/.css  # Modal chỉnh sửa dự án (V2 - với geocoding)
│   │   ├── ModalQuanLyChinhSachCoc.jsx/.css # Modal quản lý chính sách cọc
│   │   ├── ModalYeuCauMoLaiDuAn.jsx/.css # Modal yêu cầu mở lại dự án banned
│   │   ├── ModalChinhSuaToaDo.jsx     # Modal điều chỉnh tọa độ GPS
│   │   ├── AddressAutocompleteInput.jsx/.css # Input tự động suggest địa chỉ
│   │   └── README.md                  # Documentation cấu trúc components
│   ├── Layout/               # Layout components tái sử dụng
│   ├── header.jsx/.css       # Header chung
│   └── footer.jsx/.css       # Footer chung
├── layouts/
│   └── ChuDuAnLayout.jsx/.css # Layout wrapper cho module Chủ dự án
├── services/
│   └── ChuDuAnService.js     # API calls tập trung cho Chủ dự án
├── styles/
│   └── ChuDuAnDesignSystem.css # Design tokens cho Chủ dự án
├── context/                  # React Context (Auth, Role, Theme)
├── hooks/                    # Custom hooks
└── utils/                    # Utility functions
```

### Quy ước Component Naming (Frontend)
- **Page:** `TenTrang.jsx` + `TenTrang.css` trong cùng thư mục `pages/TenTrang/`
- **Component:** `TenComponent.jsx` + `TenComponent.css` trong `components/TenComponent/`
- **Layout:** `TenLayout.jsx` + `TenLayout.css` trong `layouts/`
- **Service:** `tenService.js` trong `services/`

### Cấu trúc Backend (chuẩn hóa)
```
server/
├── api/
│   ├── ChuDuAn/              # Routes cho module Chủ dự án
│   │   ├── tinDangRoutes.js         # Routes quản lý tin đăng
│   │   ├── quanLyTinDang.js         # Routes CRUD tin đăng
│   │   ├── uploadRoutes.js          # Routes upload ảnh
│   │   └── khuVucRoutes.js          # Routes quản lý khu vực
│   └── ...                   # Các module khác
├── controllers/
│   ├── ChuDuAnController.js  # Controller cho Chủ dự án (validate + call service)
│   └── userController.js     # Controller user chung
├── models/
│   ├── ChuDuAnModel.js       # Model xử lý DB logic (mysql2 queries)
│   └── userModel.js          # Model user chung
├── services/
│   └── NhatKyHeThongService.js # Audit log service (tracking actions)
├── middleware/
│   ├── auth.js               # JWT authentication
│   ├── authSimple.js         # Auth đơn giản cho dev
│   ├── role.js               # RBAC + ownership verification
│   └── roleSimple.js         # Role checking đơn giản
├── routes/
│   ├── chuDuAnRoutes.js      # Mount tất cả routes /api/chu-du-an
│   ├── tinDangRoutes.js      # Routes tin đăng
│   └── userRoutes.js         # Routes user
├── config/
│   └── db.js                 # MySQL connection pool (mysql2)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HoanhHop11/Hommy-Website](https://github.com/HoanhHop11/Hommy-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
