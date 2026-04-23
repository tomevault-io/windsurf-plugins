---
trigger: always_on
description: > **Scope:** Áp dụng cho mọi tác vụ Test Automation do Gemini (Antigravity) hoạt động trong dự án này.
---

# GEMINI AI - GLOBAL AUTOMATION AGENT RULES

> **Scope:** Áp dụng cho mọi tác vụ Test Automation do Gemini (Antigravity) hoạt động trong dự án này.
> **Mục tiêu:** Sinh ra test scripts hiệu quả, ổn định – dễ debug – dễ scale – CI friendly.

---

## 🔐 Security & Credentials (ƯU TIÊN CAO NHẤT — ĐỌC TRƯỚC)

> ⚠️ **Quy tắc này có hiệu lực ngay khi bắt đầu task. AI PHẢI đọc và tuân thủ trước khi làm bất kỳ hành động nào liên quan đến xác thực.**

### ❌ CẤM đọc file `.env` trực tiếp

AI **KHÔNG ĐƯỢC** sử dụng bất kỳ công cụ nào (`view_file`, `read_url_content`, `run_command`, `grep`, v.v.) để đọc nội dung file `.env` nhằm mục đích lấy thông tin đăng nhập (username, password, token, API key...).

> **Lý do:** File `.env` chứa credentials nhạy cảm. Việc đọc trực tiếp có nguy cơ lộ thông tin trong log, chat history hoặc artifact AI.

---

## Browser Rules (MANDATORY)

### 🖥️ Viewport & Mode

* Tất cả **UI debugging** phải chạy với **desktop viewport**: **`1920x1080`**
* Bắt buộc **mở browser thật** khi debug (headed mode)
* **Headless mode** chỉ được sử dụng **sau khi test đã debug PASS trên UI**
* CI/CD pipeline **được phép chạy headless mặc định**

### 🔄 Thứ Tự Debug Bắt Buộc (Playwright MCP)

Khi dùng Playwright MCP để debug UI, **LUÔN** tuân theo thứ tự:

```
navigate → resize(1920×1080) → wait_for(page_load) → snapshot → interact → screenshot(on_fail)
```

* **KHÔNG** gọi `browser_navigate` lại nếu đã đang ở đúng trang — tránh reload ngoài ý muốn
* **LUÔN** gọi `browser_resize(width=1920, height=1080)` ngay sau `browser_navigate`
* **LUÔN** verify page đã load xong trước khi lấy snapshot hoặc tương tác

### 📸 Screenshot & Snapshot

* Dùng **`snapshot`** để phân tích DOM và xác định locator
* Dùng **`screenshot`** để lưu bằng chứng khi test fail hoặc để báo cáo
* Chụp **screenshot ngay khi assertion fail** để hỗ trợ truy vết lỗi
* **KHÔNG** chụp screenshot tràn lan — chỉ khi cần thiết (fail / milestone quan trọng)

---

## Tools

### 🛠️ Ưu Tiên Sử Dụng

* Ưu tiên sử dụng **Playwright MCP** cho tất cả tác vụ debug UI
* Tham chiếu rule chi tiết: [Quy tắc Playwright](.agent/rules/playwright_rules.md)

### 🔍 Inspect & Debug

* Mở browser thật để debug (headed mode)
* Inspect **DOM / HTML thực tế** trên trình duyệt — **KHÔNG đoán locator**
* Execute và debug test trực tiếp trên UI trước khi sinh code
* **KHÔNG** generate code khi chưa inspect DOM

### ⚡ Nguyên Tắc

* Một locator phải được **verify chạy được** trên browser hiện tại trước khi đưa vào code
* Nếu locator lấy từ code cũ → **bắt buộc verify lại** trước khi dùng

---

## Cleanup & Delivery

### ✅ Điều kiện bàn giao (Definition of Done)

Test chỉ được coi là **hoàn thành** khi đáp ứng **toàn bộ** các tiêu chí sau:

#### 🧹 Code Cleanup

- [ ] Xoá toàn bộ `print()`, `console.log()`, debug log tạm thời
- [ ] Xoá locator không còn sử dụng
- [ ] Không để lại commented-out code
- [ ] Không có `waitForTimeout` / `Thread.sleep` hardcoded
- [ ] Không có test data hardcoded (email, username, ID phải random/traceable)

#### 🏗️ Cấu trúc & POM

- [ ] Tuân thủ mô hình **Page Object Model** — tách biệt Page class, Test class, Utils
- [ ] Locator được định nghĩa trong Page class, không viết inline trong test
- [ ] Tên file, class, method đặt theo convention rõ ràng và nhất quán
- [ ] Import không còn thừa (unused imports)

#### ✔️ Chất lượng Test

- [ ] Test **PASS ổn định** ít nhất **2 lần liên tiếp** trên UI (headed mode)
- [ ] Assertion có message rõ ràng, dễ debug khi fail
- [ ] Mỗi test case độc lập — không phụ thuộc thứ tự chạy
- [ ] Test data được sinh động (timestamp/random) và traceable

#### 📁 File Output

- [ ] Source code được lưu đúng vị trí trong project structure
- [ ] Không có file tạm, file test thừa trong thư mục source
- [ ] File cấu hình (config, .env) không chứa credentials thật

#### 📋 Báo Cáo Kết Quả

- [ ] Tóm tắt kết quả: số test PASS / FAIL / SKIP
- [ ] Nêu rõ các TC đã implement và TC nào bị skip (kèm lý do)
- [ ] Ghi chú các known issues hoặc limitation nếu có

---

## 1. Ngôn Ngữ & Giao Tiếp

- Luôn giao tiếp, giải thích ý tưởng và báo cáo bằng **Tiếng Việt**.
- Diễn giải **ngắn gọn, rõ ràng, dễ hiểu**.
- Tránh suy đoán lập trình hoặc giải thích mơ hồ về lỗi mà cần có căn cứ trực tiếp.

## 2. Quy Trình Làm Việc (Workflow)

- **Recon (Điều tra):** Luôn inspect giao diện thực tế hoặc DOM/HTML/XML trước khi viết automation. Tuyệt đối KHÔNG ĐOÁN locator.
- **Implementation:** Giữ vững mô hình **Page Object Model (POM)**. Phân tách rõ Page objects, Test execution và Utils/Test data.
- **Execution & Self-fix:** Chạy test ngay sau khi code xong. Nếu test FAIL → tự đọc log → phân tích root cause → sửa code → chạy lại → đến khi PASS ổn định. Chỉ hỏi User khi gặp business rule mâu thuẫn.
- **Cleanup:** Gỡ bỏ debug logs, code thừa, locator không dùng trước khi deliver.

## 3. Tech Stack Hỗ Trợ

| Loại             | Công nghệ                                     |
| ----------------- | ----------------------------------------------- |
| Ngôn ngữ        | Java, TypeScript                                |
| Web Automation    | Playwright (TS/Java), Selenium WebDriver (Java) |
| Mobile Automation | Appium (Java)                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anhtester/antigravity-testing-kit](https://github.com/anhtester/antigravity-testing-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
