---
trigger: always_on
description: > **Thái Lai Billiards** · Cập nhật: tháng 6/2026
---

# Bida Manager — Tài liệu Website

> **Thái Lai Billiards** · Cập nhật: tháng 6/2026

---

## Tổng quan hệ thống

```
staff.html        ──── Google Sheets API (hiện tại)
owner_desktop.html ──── Go REST API + WebSocket (mới)
                            ↓
                       PostgreSQL @ 162.4.176.129
```

Hệ thống gồm 2 file HTML độc lập, không phụ thuộc lẫn nhau, dùng chung cùng 1 cơ sở dữ liệu.

---

## 1. Staff Web (`staff.html`)

**Dành cho:** Nhân viên vận hành hàng ngày  
**Kích thước:** ~130KB, 2,306 dòng  
**Database:** Google Sheets (JSONP + no-cors fetch)  
**Realtime:** Polling 5 giây  
**URL Sheets:** `https://script.google.com/.../exec`

### 1.1 Navigation (7 tab)

| Icon | Tab | Trang | Mô tả |
|------|-----|-------|-------|
| 🎯 | Tab khách | `pageTabs` | Quản lý tab đang mở |
| 🎱 | Bida | `pageBida` | Điều khiển bàn bida |
| 📦 | Nhập hàng | `pageInventoryStaff` | Kho tồn + nhập hàng |
| 📜 | Lịch sử | `pageHistoryStaff` | Xem lịch sử (chỉ đọc) |
| 📋 | Tổng kết | `pageDailySummary` | Tổng kết ngày |
| 💳 | Nợ | `pageDebts` | Danh sách công nợ |
| 📝 | Điều chỉnh | `pageStaffAdjust` | Thêm điều chỉnh doanh thu |

### 1.2 Tính năng chi tiết

#### 🎯 Tab khách
- **Tạo tab mới** — nhập tên khách → tạo tab với `id = 't' + Date.now()`
- **⚡ Order nhanh** — chọn đồ → thanh toán ngay không cần tab
  - Thanh toán tiền mặt / QR VietQR
  - Ghi nợ với thông tin khách
- **Tab card** hiển thị: tên khách, ngày giờ, tiền bida, danh sách đồ đã order
- **Bấm vào tab** → mở modal Order (thêm/trả đồ) hoặc Thanh toán

#### 🎱 Bida

**Vòng đời bàn bida:**
```
Trống → [Bắt đầu] → Đang chạy → [Dừng] → Tạm dừng
                                              ↓ [Tiếp]
                         [Order]          Đang chạy
                            ↓
                    Cart trong bidaState
                            ↓
               [Kết thúc] → Tạo tab khách
               [Thanh toán] → Thanh toán trực tiếp
               [Gắn tab] → Gộp vào tab có sẵn
```

- `bidaState` lưu trong Google Sheets (sheet `bida_state`), persist qua reload
- **Thời gian** tính từ `startTime` (ISO string), không bị reset khi dừng/tiếp
- **Đa thiết bị:** `closedAt` field phát hiện bàn bị đóng từ thiết bị khác
- **Giá bàn** chuyển theo giờ cắt (`CUTOFF_HOUR:CUTOFF_MIN`)

| Loại bàn | ID | Trước giờ cắt | Sau giờ cắt |
|---|---|---|---|
| Bàn phân 1 | b1 | 25.000đ/h | 30.000đ/h |
| Bàn phân 2 | b2 | 25.000đ/h | 30.000đ/h |
| Bàn 3C | b3 | 40.000đ/h | 45.000đ/h |

**Order từ bida:**
- Cart lưu trong `bidaState[bid].cart` (không tạo tab tạm)
- Hỗ trợ trả/đổi món qua nút **↩ Trả/Đổi hàng**
- Khi kết thúc → gộp cart vào tab

#### 📦 Nhập hàng
- Xem tồn kho theo danh mục
- Nhập hàng: chọn mặt hàng → số lượng → tổng tiền → ghi chú nguồn hàng
- Lịch sử nhập: hiện số lượng + tiền nhập

#### 📜 Lịch sử (chỉ đọc)
- Xem sessions hôm nay và gần đây (50 bản ghi)
- Mỗi session hiển thị: tên tab, giờ, nhân viên, 🎱 tiền bida, 🛒 đồ đã order, tổng tiền
- **Không có nút sửa/xóa** (khác với admin)

#### 📋 Tổng kết ngày
- **Thu:**
  - 🎱 Tiền bida: tổng `bidaCost` các sessions hôm nay
  - 🥤 Tiền vật phẩm: tổng `cartCost` các sessions hôm nay
  - Điều chỉnh (+): tổng adjustments dương
  - Tổng thu
- **Chi:**
  - 📦 Nhập hàng: tổng `cost` các lần nhập hôm nay
  - Các điều chỉnh âm (có tên cụ thể)
  - Tổng chi
- **Doanh thu ngày** = Tổng thu − Tổng chi
- **Bảng vật phẩm:** cột Bán / Nhập / ±

#### 💳 Nợ
- Danh sách khách đang nợ
- Trạng thái: `pending` / `partial` / `paid`
- Bấm **Trả nợ** → nhập số tiền trả → cập nhật remaining
- Tìm kiếm theo tên khách

#### 📝 Điều chỉnh
- **➕ Cộng tiền:** thu thêm (khách trả bù, thu linh tinh)
- **➖ Chi phí vận hành:** mua đá, mua đồ dùng, chi phí nhỏ
- Danh sách điều chỉnh hôm nay và toàn bộ lịch sử

### 1.3 Thanh toán (modal `mo-pay`)

Flow thanh toán tab khách:
```
openPayModal(tabId)
  → renderPayBody()
     - Hiện cart (đồ đã order)
     - Thêm đồ bổ sung (extraCart)
     - Trả lại đồ (returnCart) → trừ khỏi cart trước khi tính
     - Tiền bida (nếu có)
     - Grand total = roundUp1k(bidaCost + cartCost)
  → [Thanh toán] → finalPay(grand)
  → [QR] → hiện VietQR → xác nhận
  → [Cho nợ] → openDebtFromPay(grand) → form ghi nợ
```

**Làm tròn:** `roundUp1k(n)` = làm tròn lên đến nghìn gần nhất

### 1.4 Data Layer (Google Sheets)

**Cấu trúc save:**
- `saveAll()` — debounce 2s, gửi toàn bộ data
- `saveNow(bid)` — debounce 300ms, chỉ gửi `bidaState` của 1 bàn
- `saveAllNow(bid)` — debounce 300ms, gửi toàn bộ data + bidaState của bàn đó

**Sync:**
- JSONP GET mỗi 5 giây (tránh CORS khi mở `file://`)
- So sánh `_hash` để bỏ qua nếu không có thay đổi
- `applyData()` normalize date/month từ Sheets → đúng format

**Sheets:**

| Sheet | Dữ liệu |
|-------|---------|
| `users` | Tài khoản người dùng |
| `tabs` | Tab khách đang mở |
| `sessions` | Lịch sử thanh toán |
| `expenses` | Chi phí |
| `adjustments` | Điều chỉnh doanh thu |
| `invImports` | Lịch sử nhập hàng |
| `inventory` | Kho hàng |
| `debts` | Công nợ |
| `settings` | Cài đặt (giá, giờ cắt) |
| `bida_state` | Trạng thái bàn bida |

### 1.5 Modals

| Modal ID | Chức năng |
|----------|-----------|
| `mo-order` | Order thêm / trả đồ |
| `mo-pay` | Thanh toán |
| `mo-qr` | Hiển thị QR VietQR |
| `mo-newtab` | Tạo tab mới / Order nhanh |
| `mo-debt` | Ghi nợ |
| `mo-inv` | Nhập hàng |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThanhDucNguyen/bida-api](https://github.com/ThanhDucNguyen/bida-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
