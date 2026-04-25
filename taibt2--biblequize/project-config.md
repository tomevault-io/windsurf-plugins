---
trigger: always_on
description: - Không bao giờ sửa code module khác khi đang làm 1 module
---

# Project: BibleQuiz

## Nguyên tắc tuyệt đối
- Không bao giờ sửa code module khác khi đang làm 1 module
- Mỗi thay đổi phải có test pass trước khi commit
- Không tự ý thêm dependency mới — hỏi trước
- Ưu tiên đọc TODO.md trước khi làm bất cứ thứ gì
- LUÔN chia nhỏ task vào TODO.md TRƯỚC khi code — không tự xử lý 1 lần

## Quy trình quản lý Task (BẮT BUỘC)

### Nguyên tắc cốt lõi
> **KHÔNG BAO GIỜ nhận prompt rồi chạy hết 1 lần.**
> Phải chia nhỏ → ghi TODO.md → làm từng task → check ✅ → task kế.

### Quy trình khi nhận prompt/task mới

```
1. ĐỌC TODO.md hiện tại → xem có task đang dở không
2. Nếu có task dở → hoàn thành nó trước (test pass, đánh ✅)
3. PHÂN TÍCH prompt mới → chia thành các task nhỏ (mỗi task 1 commit)
4. GHI tất cả tasks vào TODO.md theo format bên dưới
5. BẮT ĐẦU task đầu tiên
6. Xong task → chạy test → pass → đánh ✅ trong TODO.md → commit
7. Sang task tiếp theo → lặp lại bước 6
8. Hết tasks → chạy full regression → cập nhật TODO.md
```

### Format TODO.md

```markdown
## [Version/Phase] — [Tên nhóm task] [IN PROGRESS/DONE]

### Task [number]: [Tên task ngắn gọn]
- Status: [ ] TODO / [x] DONE / [!] BLOCKED
- File(s): [danh sách files sẽ sửa]
- Test: [test file tương ứng]
- Checklist:
  - [ ] Sub-step 1
  - [ ] Sub-step 2
  - [ ] Unit test pass
  - [ ] Commit: "[type]: [message]"
```

### Ví dụ cụ thể

Khi nhận prompt "Sync Home Dashboard từ Stitch + fix bugs + viết tests":

KHÔNG LÀM: đọc prompt → code hết 1 lần → commit 1 cục

LÀM ĐÚNG: chia thành TODO rồi làm từng task:
```markdown
## v2.5 — Sync Home Dashboard [IN PROGRESS]

### Task 1: Đọc Stitch design qua MCP + diff với code
- Status: [ ] TODO
- File(s): DESIGN_SYNC_AUDIT.md (output)
- Checklist:
  - [ ] MCP query Home screen
  - [ ] Liệt kê diff points
  - [ ] Ghi kết quả

### Task 2: Fix UNDEFINED energy bug
- Status: [ ] TODO
- File(s): GameModeGrid.tsx
- Test: GameModeGrid.test.tsx
- Checklist:
  - [ ] Xác định root cause (field name mismatch)
  - [ ] Fix code
  - [ ] Unit test cho null/undefined/loading cases
  - [ ] Vitest pass
  - [ ] Commit: "fix: energy UNDEFINED in GameModeGrid"

### Task 3: Cập nhật Welcome Bar theo Stitch
- Status: [ ] TODO
- File(s): Home.tsx
- Test: Home.test.tsx
- Checklist:
  - [ ] Compact layout (80px)
  - [ ] Greeting logic (sáng/chiều/tối)
  - [ ] Tier progress bar
  - [ ] Unit test
  - [ ] Vitest pass
  - [ ] Commit: "sync: Home welcome bar from Stitch"

### Task 4: Cập nhật Game Mode Grid theo Stitch
...

### Task 5: Cập nhật Leaderboard Preview
...

### Task 6: Fix warnings (useEffect → useQuery)
...

### Task 7: Full regression
- Status: [ ] TODO
- Checklist:
  - [ ] npx vitest run
  - [ ] npx playwright test
  - [ ] Backend tests
  - [ ] Số test >= baseline
```

### Rules
- Mỗi task phải ĐỦ NHỎ để commit riêng (1 task = 1 commit)
- KHÔNG gộp nhiều thay đổi vào 1 task
- KHÔNG skip task, làm theo thứ tự
- KHÔNG bắt đầu task kế nếu task hiện tại chưa ✅
- Sau mỗi task: cập nhật TODO.md NGAY (đánh ✅, ghi commit hash nếu cần)
- Nếu task bị BLOCKED: ghi lý do, chuyển sang task không phụ thuộc, quay lại sau

## Stack
- Backend: Spring Boot 3.3 (Java 17), port 8080
- Frontend: Vite 5 + React 18 + TypeScript 5.4, port 5173
- DB: MySQL 8.0 (Docker, port 3307)
- Cache: Redis 7 (Docker, port 6379)
- Unit Test: Vitest 4.1 (happy-dom) + @testing-library/react
- E2E Test: Playwright (Chromium, baseURL localhost:5173)
- Design: Stitch MCP (project ID `5341030797678838526`)

## Quản lý quyết định

Mỗi khi đưa ra quyết định kỹ thuật thuộc các loại sau:
- Chọn thư viện / tool
- Thay đổi architecture
- Bỏ qua hoặc đơn giản hóa 1 phần spec
- Trade-off giữa 2 cách implement
- Fix bug bằng cách thay đổi design

→ Tự động ghi vào DECISIONS.md theo format:
```
## YYYY-MM-DD — [Tiêu đề ngắn]
- Quyết định: [làm gì]
- Lý do: [tại sao]
- Trade-off: [đánh đổi gì]
- KHÔNG thay đổi khi refactor trừ khi có lý do mới
```

## Local Dev Start
```bash
docker compose up -d mysql redis          # 1. Infra
cd apps/api && ./mvnw spring-boot:run     # 2. Backend (terminal 1)
cd apps/web && npm run dev                # 3. Frontend (terminal 2)
```

## Quy tắc bắt buộc
1. Sau mỗi thay đổi code → chạy test ngay (xem quy trình test bên dưới)
2. Nếu test fail → tự fix → chạy test lại, lặp đến khi pass
3. Không hỏi xác nhận, tự quyết định
4. Không dừng giữa chừng trừ khi có lỗi không thể tự fix
5. KHÔNG commit nếu full regression chưa pass
6. Phát hiện regression → DỪNG feature mới → fix regression → pass hết → mới được tiếp tục

## Quy trình test bắt buộc (Regression Guard)

### Nguyên tắc cốt lõi
> **Mỗi dòng code thay đổi đều có thể phá vỡ chức năng khác.**
> Chạy test đơn lẻ chỉ chứng minh code mới hoạt động.
> Chạy full regression chứng minh code mới KHÔNG phá code cũ.

### 3 tầng test — chạy theo thứ tự, KHÔNG được bỏ tầng nào

**Tầng 1 — Scope Test (sau mỗi thay đổi nhỏ, trong khi code)**
```bash
# Chỉ chạy test của file/module vừa sửa → feedback nhanh
cd apps/web && npx vitest run src/pages/Home.test.tsx        # FE unit
cd apps/api && ./mvnw test -Dtest="XxxServiceTest"           # BE unit
```
- Mục đích: kiểm tra code vừa viết hoạt động đúng
- Khi nào: sau mỗi function/component hoàn thành

**Tầng 2 — Related Test (sau khi hoàn thành 1 screen/feature)**
```bash
# Chạy test của các module liên quan có thể bị ảnh hưởng

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TaiBT2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
