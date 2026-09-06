---
trigger: always_on
description: > **Sứ mệnh:** Nền tảng mã nguồn mở kết nối Zalo cá nhân với Chatwoot CRM, quản lý hội thoại Live Chat, chiến dịch Remarketing và thẻ tag.
---

# 🤖 AGENTS.MD — QUY CHUẨN VẬN HÀNH & PHÁT TRIỂN ZALO-FLOW

> **Sứ mệnh:** Nền tảng mã nguồn mở kết nối Zalo cá nhân với Chatwoot CRM, quản lý hội thoại Live Chat, chiến dịch Remarketing và thẻ tag.
> **Mục đích:** Nghiên cứu kỹ thuật, học tập kiến trúc và tự động hóa cá nhân (Educational & Research Only).
> **Chính sách:** Nghiêm cấm tuyệt đối mọi hành vi Spam, quấy rối hoặc thu thập dữ liệu trái phép.

---

## 🏛️ 1. Thông Số Dự Án Cốt Lõi

| Hạng Mục | Thông Số Kỹ Thuật |
| :--- | :--- |
| **Tên dự án** | Zalo-Flow (AIzalo Community Edition) |
| **Giấy phép** | MIT License with Commons Clause (Non-Commercial, AS-IS Disclaimer) |
| **Mô hình phục vụ** | Single-Tenant (1 tài khoản Zalo cá nhân / 1 instance thử nghiệm) |
| **Runtime** | Node.js >= 22.5.0 (ES Modules) |
| **Core Dependency** | `zca-js: 2.1.0` (Khóa cứng version) |
| **Web Server** | Express.js (Port 3000) |
| **Mã hóa Session** | AES-256-CBC với khóa `SESSION_SECRET` (Lưu tại `sessions/*.enc`) |

---

## 🛡️ 2. Ma Trận 8 Hard Guardrails Bắt Buộc (Core Invariants)

1. **Pháp Lý, Phi Thương Mại & Anti-Spam (Educational & Research Defense):** Mọi tài liệu và code BẮT BUỘC duy trì tuyên bố *"Phần mềm chỉ phục vụ mục đích học tập/nghiên cứu cá nhân, phi thương mại"*. Nghiêm cấm viết hoặc hỗ trợ bất kỳ tính năng spam tin nhắn hàng loạt, bán lại thương mại hoặc thu thập dữ liệu trái phép nào.
2. **Air-Gapped IP & Zero Secret Leak:** Tuyệt đối KHÔNG import mã nguồn, Cloudflare bindings, D1 schemas, IP VPS (`160.187.*`, `43.134.*`, `5.231.*`) hoặc credentials nội bộ từ dự án SaaS `Zalo-Bridge`. Mọi tệp `.env.example` phải dùng 100% placeholder dummy data.
3. **Lightweight Memory Footprint (< 100MB RAM):** Không cài đặt các thư viện nặng nề không cần thiết. Giữ cho bản Core luôn chạy mượt mà trên môi trường RAM < 128MB.
4. **Anti-Ban 3 Lớp Bất Biến:** Mọi tin nhắn gửi đi BẮT BUỘC đi qua `RateLimiter` (giãn cách >= 3s, tối đa 20 tin/phút) và `SelfEchoShield` (30s buffer chống loop). Mọi tin nhắn đến phải qua `FloodDetector`.
5. **Adapter Pattern Isolation:** Mọi nền tảng tích hợp mới (Dify, Make, Telegram...) BẮT BUỘC kế thừa từ `src/adapters/base-adapter.js` và xử lý lỗi độc lập, không làm sập tiến trình chính.
6. **Session Encryption Mandatory:** Không bao giờ lưu cookie/token Zalo dưới dạng plaintext. Luôn sử dụng `saveEncryptedSession()` và `loadEncryptedSession()`.
7. **In-Thread Reply Discipline:** Mặc định chỉ trả lời vào cuộc trò chuyện có sẵn (`threadId`), không chủ động gửi tin nhắn lạnh (cold outbound) tới số lạ nếu chưa có tương tác để tránh bị Zalo quét spam.
8. **Quality Gate Verification:** Trước khi commit hoặc tạo PR, BẮT BUỘC chạy `node test/test-all.js` để đảm bảo 100% unit tests và security scanner đều vượt qua.

---

## ⚡ 3. Lệnh Thao Tác Nhanh (Quick Commands)

* **Chạy kiểm thử toàn diện:** `npm test`
* **Khởi chạy môi trường Dev:** `npm run dev`
* **Chạy Wizard cấu hình tương tác:** `npm run init`
* **Khởi chạy container Docker:** `docker compose up -d`
* **Xem logs trực tiếp:** `docker compose logs -f zalo-flow`

---

## 🛡️ 4. Quy Chuẩn Kỹ Thuật Bổ Sung (Learned Standards)

9. **Universal Schema Reconciliation (Idempotent SQLite Migrations):** Không chỉ dựa vào tuyến tính `user_version`. Khi khởi tạo LocalStore, bắt buộc dùng `PRAGMA table_info` quét và bổ sung cột còn thiếu để chống lỗi schema drift trên máy người dùng.
10. **Data Field Contract (Message Text Access):** Cột nội dung chữ của tin nhắn trong SQLite luôn là `text`. Khi trích xuất hội thoại hoặc làm mẫu Few-Shot, luôn dùng `m.text || m.content`.
11. **Multer Buffer & Temp File Cleanup:** Mọi endpoint nhận upload file/ảnh qua multer phải giới hạn <= 10MB và bắt buộc dọn dẹp bằng `fs.unlinkSync` trong `finally` block để duy trì RAM < 100MB.
12. **In-Thread Forward Discipline:** Tính năng chuyển tiếp (Forward) chỉ được phép gửi tới các contact/group đã có trong bảng `conversations`, tuyệt đối không gửi tới ID lạ chưa có tương tác.
13. **Zalo Message Dual-ID Binding (`msgId` & `cliMsgId`):** Trong giao thức Zalo (`zca-js`), mọi tin nhắn đều có 2 ID (`msgId` và `cliMsgId`). Các hành động tương tác (Reactions, Undo/Recall, Quote) BẮT BUỘC phải lưu và truyền đúng cả hai (`dest.data.msgId` và `dest.data.cliMsgId`), tuyệt đối không gán `cliMsgId = msgId` vì sẽ khiến ứng dụng Zalo trên điện thoại không thể ánh xạ và hiển thị tương tác.
14. **High-Frequency Inbound Batching (Delivery Events):** Sự kiện `delivered_messages` có tần suất cao BẮT BUỘC phải gom nhóm trong bộ đệm `Map<threadId, Set<msgId>>` và xả định kỳ mỗi 3s (`flushDeliveredBuffer`) thay vì cập nhật từng tin nhắn đơn lẻ để tránh nghẽn SQLite và quá tải SSE.
15. **Frontend Vanilla JS Pre-flight AST Validation:** Sau mỗi lần chỉnh sửa tệp `public/app.js`, Agent BẮT BUỘC phải chạy lệnh kiểm tra cú pháp `node --check public/app.js` và `npm test` để phát hiện và ngăn chặn mọi lỗi `SyntaxError` (như khai báo trùng lặp biến/hàm) làm sập trình duyệt.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aizaloapp/zalo-flow](https://github.com/aizaloapp/zalo-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
