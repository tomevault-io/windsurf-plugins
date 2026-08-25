---
trigger: always_on
description: ระบบจัดการแผงตลาดสดรายวัน แก้ pain point: การแย่งจองแผงขาจร (concurrency) + การเช็คชื่อหน้างาน (roll-call) ว่าใครมา/ไม่มาตรงกับระบบ
---

# AGENTS.md — TaladSync

ระบบจัดการแผงตลาดสดรายวัน แก้ pain point: การแย่งจองแผงขาจร (concurrency) + การเช็คชื่อหน้างาน (roll-call) ว่าใครมา/ไม่มาตรงกับระบบ

> **เก็บเงิน 20 บาท:** เจ้าหน้าที่เก็บสดหน้างานเอง — **ไม่ track ในระบบ MVP** (ย้ายไป Phase 2 ถ้าต้องการ financial summary)

## เอกสารอ้างอิง (อ่านก่อนเริ่ม)

- [docs/SYSTEM_DESIGN.md](docs/SYSTEM_DESIGN.md) — ภาพรวม, stack, schema (attendance roll-call MVP)
- [docs/BOOKING_FLOW.md](docs/BOOKING_FLOW.md) — state machine, transaction strategy, concurrent test

---

## Implementation Checkpoint (อัปเดตล่าสุด)

**สถานะรวม:** Backend MVP **Phase 0–8 DONE**. Frontend **F1–F4 DONE** (merged → `main`).
**กำลังทำ: UI redesign บน branch `bookingvendor`** (push แล้ว)

- `ensureWalkin` → refactor เป็น `ensureRole(role)` (auth store เก็บ role คู่ token กัน 403 ข้าม role)
- **DevNav ถูกลบแล้ว** (commit `9698a12`) — เข้าแต่ละหน้าด้วย URL ตรงๆ (`/`, `/staff`, `/regular`, `/directory`)

### 🎨 UI Redesign — branch `bookingvendor` (กำลังทำ)

เพื่อนทำ UX/UI ออกแบบให้ · ทำ**ทีละ component** แล้วให้ user review ก่อน commit

**เสร็จแล้ว** (commit `5cce091`):

| ไฟล์ | ทำอะไร |
|------|--------|
| `components/PageHeader.vue` 🆕 | header ตามดีไซน์: ปุ่ม `<` (absolute ซ้าย) + title กลางจอ · emit `back` |
| `views/WalkinView.vue` | list → **กริด 3 คอลัมน์ แบ่งตามโซน** (`computed stallsByZone`) + พื้นหลัง `bg-brand-50` |
| `components/StallCard.vue` | list row → **การ์ดกริด** 3 สถานะ (ว่าง / selected / disabled) |
| `database/seed.go` | **โซน A (A-01..A-15 มีเจ้าของ) + โซน B (B-01..B-15 ว่าง)** แทน Zone A + Pool เดิม |

**⚠️ สำคัญ:** `AppHeader.vue` **ยังใช้อยู่** ในอีก 3 หน้า (staff/regular/directory) — **ห้ามลบ** จนกว่าจะ redesign ครบทุกหน้า

**🔴 ติดรอดีไซน์จากเพื่อน (ทำต่อไม่ได้):**
1. **Flow การจอง** — กดแผงแล้วเกิดอะไร? (เลือกก่อนค่อยกดปุ่ม vs เด้ง modal ทันที) · ตอนนี้คงของเดิม = กดแล้วเด้ง `BookingModal`
2. **สถานะ "เลือกแล้ว"** หน้าตายังไง (ตอนนี้เดาไว้: `bg-brand-50` + `ring-2 ring-brand-500`)
3. **ปุ่ม "จองแผง"** ข้างล่าง (ขึ้นกับข้อ 1)
4. ดีไซน์อีก 3 หน้า + `BookingModal`

### Next session — ทำได้เลย (ไม่ต้องรอดีไซน์)

1. **Vitest test ตัวแรก** — frontend ยังไม่มี test (ปิดจุดอ่อนตอนสัมภาษณ์ + อยู่ใน roadmap)
2. ~~**BOOK audit**~~ — **DONE**: จองสำเร็จเขียน `audit_logs` action `BOOK` แล้ว (ครบทั้ง BOOK/LEAVE/CHECKIN/NOSHOW)
3. **CI/CD** — GitHub Actions รัน `go test` + `npm run type-check` ทุก push
4. **Deploy จริง** — Railway/Render → มี URL ใส่ resume
5. ใช้ design guidance ใน `frontend/SKILL.md` ตอนสร้าง UI ใหม่

### Backend เพิ่มระหว่าง F1 (commit แล้ว `59eced9`)

| งาน | ไฟล์ | หมายเหตุ |
|------|------|----------|
| `GET /api/bookings/mine?date=` | `handler/booking.go`, `usecase/booking.go`, `repository/booking.go`, `main.go` | WALKIN เช็กจองวันนี้แล้วหรือยัง → `{ booking: null \| DailyBooking }` |
| CORS | `cmd/api/main.go` | `AllowOrigins` จาก `CORS_ORIGINS` (default `http://localhost:5173`) |
| Seed today | `database/seed.go` | `SeedDailyBookingsForToday` รันทุกครั้งที่ API start |

### Phase 8 — DONE (commit แล้ว)

| ไฟล์ | งาน |
|------|-----|
| [`docker-compose.yml`](docker-compose.yml) | `postgres:17-alpine` + `api`, secrets จาก `.env` |
| [`backend/Dockerfile`](backend/Dockerfile) | multi-stage Go 1.24 → alpine, `TZ=Asia/Bangkok` |
| [`backend/.dockerignore`](backend/.dockerignore) | กัน `.env`, binaries |
| [`.env.example`](.env.example) | template secrets (commit ได้) — copy → `.env` |

**Secrets:** compose ใช้ `${POSTGRES_PASSWORD}`, `${JWT_SECRET}` จาก `.env` (root) — **ไม่ hardcode ใน repo**

**สองไฟล์ env:**

| ไฟล์ | ใช้เมื่อ |
|------|---------|
| `.env` (root) | `docker compose up` |
| `backend/.env` | `go run ./cmd/api` |

**Pitfalls Docker:** เปลี่ยน `POSTGRES_PASSWORD` หลัง volume สร้างแล้ว → `docker compose down -v` แล้ว up ใหม่; port 5432/8080 ชน local ได้

### Incremental roadmap

| Phase | สถานะ | เป้าหมาย |
|-------|--------|----------|
| 0 | **DONE** | Health API — `cmd/api/main.go` + `GET /health` |
| 1 | **DONE** | Domain models (4 tables) + Postgres connect + migrate |
| 2 | **DONE** | Seed — **1 แม่ค้าประจำ : 1 แผง** (12 REGULAR เจ้าของ A-01..A-12 คนละแผง) + 1 WALKIN + 1 STAFF + pool P-01..P-08; mock-login REGULAR = คนแรก (เจ้าของ A-01) |
| 3 | **DONE** | Auth mock-login + JWT middleware + `GET /api/auth/me` |
| 4 | **DONE** | Booking — list/book, 409, `date?` body, partial unique index (1 vendor/วัน), unit tests |
| 5 | **DONE** | Vendor leave + staff checklist/attendance/summary + audit (LEAVE/CHECKIN/NOSHOW) |
| 6 | **DONE** | Directory — `GET /api/directory?q=` public, ไม่มี PII |
| 7 | **DONE** | Concurrent test — 50 goroutines, 1 winner (`-tags=integration`) |
| 8 | **DONE** | DevOps — compose + Dockerfile + `.env.example` |

### Frontend roadmap

| Phase | สถานะ | เป้าหมาย |
|-------|--------|----------|
| F1 | **DONE** | WALKIN — auto mock-login, รายการแผงว่าง, ฟอร์มจอง, 409 ไทย, จองแล้วแสดงการ์ด |
| F2 | **DONE** | STAFF — checklist roll-call (progress + toggle มา/ไม่มา), summary นับ client-side |
| F3 | **DONE** | REGULAR — `GET /api/vendor/mine` แผงตัวเอง + แจ้งหยุด (modal ยืนยัน, optimistic) — branch `regular-frontend` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KunachRatchapat/TaladSync](https://github.com/KunachRatchapat/TaladSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
