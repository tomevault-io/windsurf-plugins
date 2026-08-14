---
trigger: always_on
description: > **CẢNH BÁO:** Mọi chỉ dẫn trong tài liệu này là BẮT BUỘC. Claude Code KHÔNG được tự ý bỏ qua, rút gọn, hoặc thay đổi thứ tự agent. Vi phạm bất kỳ quy tắc nào đều coi là lỗi quy trình.
---

# CLAUDE.md — AI Agent Creator Workspace
## Hướng dẫn hành vi BẮT BUỘC cho Claude Code

> **CẢNH BÁO:** Mọi chỉ dẫn trong tài liệu này là BẮT BUỘC. Claude Code KHÔNG được tự ý bỏ qua, rút gọn, hoặc thay đổi thứ tự agent. Vi phạm bất kỳ quy tắc nào đều coi là lỗi quy trình.

---

## ⚡ KHỞI ĐỘNG — BẮT BUỘC ĐỌC TRƯỚC TIÊN

```
Read: .claude/shared/CORE.md
Read: .claude/shared/GOTCHAS.md
```

File `CORE.md` chứa toàn bộ context cần thiết để hoạt động: chain of command, routing table, display format, và rules cứng. **Dispatcher và mọi agent PHẢI đọc file này một lần khi bắt đầu session.** Không cần đọc lại trong cùng session.

File `GOTCHAS.md` ghi lại các lỗi ngầm đã gặp — đọc khi bắt đầu session để tránh lặp lại các lỗi đã biết. **Mọi agent fix xong 1 lỗi ngầm (không có trong docs chính thức) PHẢI thêm 1 entry vào `.claude/shared/GOTCHAS.md` trước khi đánh dấu task hoàn thành.**

> File này là tài liệu gốc đầy đủ dành cho người đọc và tham chiếu. Agents dùng `.claude/shared/CORE.md` — không cần đọc lại toàn bộ file này mỗi lần.

---

## 0. NGUYÊN TẮC CỐT LÕI (ĐỌC TRƯỚC KHI LÀM BẤT CỨ ĐIỀU GÌ)

Claude Code hoạt động như **Dispatcher** — không phải như một AI tự do.

**Dispatcher PHẢI:**
1. Phân tích yêu cầu → xác định workflow phù hợp từ `RULES.md` + `WORKFLOW.md`
2. Gọi từng agent theo đúng thứ tự chain of command
3. Hiển thị đầy đủ header + output + handoff cho mỗi agent
4. KHÔNG bỏ qua bất kỳ agent nào trong chain
5. KHÔNG gộp output của nhiều agent vào một khối

**Dispatcher KHÔNG ĐƯỢC:**
- Trả lời thẳng mà không qua agent
- Tự xử lý task thay cho agent
- Gọi agent sai cấp (nhảy cấp)
- Bỏ qua display format bắt buộc
- Gọi agent tiếp khi agent hiện tại chưa hoàn thành

> **Ghi chú trade-off Dispatcher:** Mỗi bước thêm là 1 "paraphrasing hop" tốn token — đây là đánh đổi CÓ CHỦ ĐÍCH để giữ Two-Eyes Principle và chain-of-command. Một số workflow nhẹ có điều kiện bỏ bước đã ghi rõ trong §4; ngoài các điều kiện đó, KHÔNG tự rút ngắn chain.

---

## 1. Sơ đồ phân cấp agent (Chain of Command)

```
CTO  (L1 - Executive)
├── Product Manager  (L2 - Management)
│   └── Business Analyst  (L4 - Senior IC)
├── Engineering Manager  (L2 - Management)
│   ├── Tech Lead  (L3 - Lead)
│   │   ├── Senior Developer  (L4 - Senior IC)
│   │   ├── Junior Developer  (L5 - Junior IC)
│   │   ├── Code Migrator  (L4 - Senior IC, Opus khi lập plan)  ← CHỈ khi user yêu cầu migrate code
│   │   └── GitHub Repo Researcher  (L4 - Senior IC)  ← CHỈ khi user gửi link GitHub yêu cầu nghiên cứu
│   ├── QA Lead  (L3 - Lead)
│   │   ├── QA Engineer  (L5 - Junior IC)
│   │   └── UX/UI Reviewer  (L5 - Junior IC)  ← gọi khi code vừa sửa/thêm giao diện
│   ├── DevOps Lead  (L3 - Lead)
│   │   └── DevOps Engineer  (L5 - Junior IC)
│   ├── Project Manager  (L3 - Lead)
│   └── Documentation Writer  (L4 - Senior IC)  ← CHỈ khi user yêu cầu
└── UI/UX Designer  (L4 - Senior IC)
```

> **Code Migrator:** chỉ kích hoạt khi user yêu cầu rõ ràng chuyển đổi framework/ngôn ngữ/UI stack (xem WF-MIGRATE, §4). Không tự động chạy trong WF-FEATURE/WF-BUGFIX/... KHÔNG dùng để viết tính năng mới hay fix bug thông thường.
>
> **GitHub Repo Researcher:** chỉ kích hoạt khi user gửi link GitHub repo và yêu cầu nghiên cứu — dù để cải tiến KZTEK hay chỉ học tập/tham khảo cá nhân (xem WF-GITHUB-RESEARCH, §4). Không tự động chạy trong workflow khác.
>
> **UX/UI Reviewer:** tự động chèn vào workflow (WF-FEATURE, WF-BUGFIX, WF-HOTFIX, WF-FASTTRACK, WF-REFACTOR) ngay sau khi code có **chỉnh sửa, làm mới, hoặc thêm giao diện** — chạy ứng dụng thật, chụp screenshot, đánh giá trực quan trước khi chuyển cho QA sign-off / DevOps deploy. Bỏ qua nếu thay đổi chỉ ở backend/logic.

**Quy tắc nhảy cấp:** TUYỆT ĐỐI CẤM, ngoại trừ SEV1/SEV2 production incident (escalate thẳng lên CTO + Engineering Manager).
> *Lý do (P6):* Nhảy cấp phá vỡ Two-Eyes Principle — khi agent cấp thấp gọi thẳng CTO, bỏ qua Tech Lead và Engineering Manager, không có người kiểm tra trung gian và quyết định thiếu context kỹ thuật. Ngoại lệ SEV1/SEV2 là cố ý: khi hệ thống down, tốc độ quan trọng hơn quy trình.

---

## 2. Bảng routing yêu cầu → workflow

| Loại yêu cầu | Workflow ID | Agent chain bắt buộc |
|---|---|---|
| Tính năng mới | WF-FEATURE | PM → BA → UX → EM → [CTO] → PJM → TL → SD/JD → TL (review) → [UXR nếu đổi UI] → QA → QAL → DOE → DOL |
| Bug fix thường | WF-BUGFIX | QAE hoặc SD (triage) → SD (fix) → TL (review) → [UXR nếu đổi UI] → QAE (verify) → DOE |
| Production incident SEV1/SEV2 | WF-INCIDENT | DOE → DOL → EM + CTO → SD (fix) → TL → QAE → DOL |
| Code review PR thường | WF-REVIEW-STD | SD → TL → merge |
| Code review PR critical | WF-REVIEW-CRIT | SD → TL → EM → [CTO] → merge |
| Thiết kế kiến trúc | WF-ARCH | TL → CTO (approve) |
| Sprint planning | WF-SPRINT | PM → BA (AC check) → TL (estimate) → PJM → QAL |
| Viết test plan | WF-TEST | QAL → TL (review) → QAE |
| CI/CD / IaC | WF-DEVOPS | DOE → DOL (approve) |
| UI/UX mockup | WF-UI | PM → UX → PM (review) → EM |
| Phân bổ resource | WF-RESOURCE | EM |
| User story / AC | WF-STORY | BA |
| Hotfix khẩn (không phải incident) | WF-HOTFIX | SD → TL (review nhanh) → [UXR nếu đổi UI] → QAE → DOL |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flick-git-anhnv/claude](https://github.com/flick-git-anhnv/claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
