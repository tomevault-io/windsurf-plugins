---
trigger: always_on
description: > **Project:** `invest-agents` (Trinity-Wealth-Engine)
---

# CLAUDE.md — กฎเหล็กและคู่มือสถาปัตยกรรมระบบ (Trinity Wealth Engine)

> **Project:** `invest-agents` (Trinity-Wealth-Engine)
> **Scope:** เอกสารฉบับนี้เป็น "กฎเหล็กและแนวทางปฏิบัติเชิงวิศวกรรม" สำหรับ Claude Code และ AI Coding Agent ในการพัฒนา ดูแลรักษา และปรับปรุงระบบ
> **Core Principle:** ยึดมั่นในความถูกต้องทางการเงิน (Financial Integrity), สถาปัตยกรรม Hexagonal Architecture, และเสถียรภาพระดับ Production (High Resilience & Distributed Safety)

---

## 1. ⚙️ สภาพแวดล้อมและคำสั่งหลัก (Environment & CLI Commands)

* **ระบบปฏิบัติการ:** Windows
* **เทคโนโลยีหลัก:** Python 3.11+, FastAPI, React 19, Vite, TypeScript, SQLite, LangGraph/LangChain
* **การจัดการ Dependency:** `uv` (Fast Python Package Manager) และ `npm`

### 🚀 คำสั่งจัดการระบบ
* **ติดตั้ง / ซิงค์ Python Dependencies:**
  ```powershell
  uv sync
  ```
* **รัน Backend Server (FastAPI + Outbox Workers):**
  ```powershell
  .venv\Scripts\python -m uvicorn api.main:app --port 8000 --reload
  ```
* **รัน Frontend Dev Server (React + Vite):**
  ```powershell
  npm --prefix web run dev
  ```
* **รัน Automated Test Suites:**
  * Backend Pytest (สถาปัตยกรรม, Unit, Integration):
    ```powershell
    .venv\Scripts\pytest tests/ -v
    ```
  * Frontend Vitest:
    ```powershell
    npm --prefix web test -- --run
    ```
  * Frontend TypeScript Check:
    ```powershell
    npm --prefix web run typecheck
    ```
* **สร้าง / ปรับปรุง Golden Baseline Manifests:**
  ```powershell
  .venv\Scripts\python scripts/generate_golden_manifests.py
  ```

---

## 2. 🧠 Mandatory Plan-First Workflow (กฎเหล็ก: วางแผนก่อนลงมือทำ)

1. **ห้ามแก้ไขไฟล์ทันที:** เมื่อได้รับโจทย์ที่ซับซ้อน (สร้างฟีเจอร์, ปรับสถาปัตยกรรม, แก้ไขบั๊กข้ามชั้น) **ห้าม** แตะต้องไฟล์โค้ดทันที
2. **เสนอแผนงานก่อนเสมอ (Execution Plan):** ร่างแผนการทำงานทีละขั้นตอน (Step-by-Step) เป็นภาษาไทย โดยระบุ:
   - 📂 ไฟล์ที่จะแก้ไขหรือสร้างใหม่ (ระบุ Path ชัดเจน)
   - ⚙️ Logic หรือ Interface ที่จะเปลี่ยนแปลง
   - ⚠️ ผลกระทบต่อ Layer อื่น และความเสี่ยง (Risks & Mitigations)
   - 🧪 แผนการทดสอบเพื่อยืนยันความถูกต้อง (Verification Plan)
3. **รอการอนุมัติ (Wait for Approval):** หยุดรอจนกว่าผู้ใช้จะอนุมัติ ("Approve", "OK", "เห็นด้วย", "ลุยเลย")
4. **ลงมือทำแบบศัลยกรรม (Surgical Changes):** แก้ไขเฉพาะจุดที่จำเป็น เคารพสไตล์เดิม ไม่ refactor โค้ดรอบข้างโดยไม่จำเป็น และไม่ทิ้ง Dead Code

---

## 3. 🏛️ สถาปัตยกรรม Hexagonal Architecture & 16 AST Rules

ระบบถูกตรวจสอบความถูกต้องเชิงสถาปัตยกรรมอัตโนมัติผ่าน 16 AST Rules ใน `tests/architecture/test_dependency_rules.py` โดยมีโครงสร้างแบ่งแยกชั้นดังนี้:

```text
[ Inbound Routers (api/routers/) ]    [ Background Workers (api/workers/) ]
                  │                                     │
                  ▼                                     ▼
         [ Application Services / Ports / DTOs (application/) ]
                                  │
                                  ▼
                    [ Domain Entities / Core Logic (core/) ]
                                  ▲
                                  │
[ Driven Adapters: Obsidian Vault | SQLite DB | LLMs | Data APIs (tools/ | api/db/) ]
```

### 3.1 กฎการแยก Layer (Layer Isolation Rules)
1. **Domain Layer (`core/`, `domain/`):** เป็นศูนย์กลางของ Business Rules ห้าม Import ชั้นนอก (Infrastructure, Application, API, หรือ Adapters) เด็ดขาด
2. **Application Layer (`application/`):** จัดการ Use Case Workflows, Ports (Interfaces), DTOs และ Saga Orchestrators **ห้าม Import Database Direct Query, SQLite Connection หรือ HTTP Frameworks ตรงๆ**
3. **Driven Adapters (`tools/`, `infrastructure/`):** ทำหน้าที่ Implement Ports ที่ Application กำหนด (เช่น `ObsidianEarningsCallAdapter`, `SqliteEarningsCallWorkflowAdapter`)
4. **Inbound Adapters (`api/routers/`):** ทำหน้าที่รับ HTTP Request, Validate ข้อมูล และส่งต่อให้ Application Service **ห้ามทำ Filesystem I/O ใน Router เด็ดขาด**
5. **DAO Pattern (`api/db/repositories/`):** ห้าม DAO เรียก `.commit()` หรือ `.rollback()` เองภายในฟังก์ชัน การเปิด-ปิด Transaction ต้องถูกควบคุมในระดับ Service หรือ Unit of Work
6. **Background Workers (`api/workers/`):** ต้องทำงานผ่าน Application Services และ Outbox Repositories เท่านั้น ห้ามเรียก State DB โดยตรง

---

## 4. 🔄 Distributed Safety: Saga & Transactional Outbox Pattern

เมื่อมีกระบวนการที่ต้องทำงานข้ามระบบที่ไม่สามารถ Commit ใน Transaction เดียวกันได้ (เช่น Obsidian Markdown Vault + LLM API + SQLite Database / Kanban):

### 4.1 Saga State Machine & Transactional Outbox
* **Saga States:** `NEW` $\rightarrow$ `SUMMARIZED` $\rightarrow$ `NOTE_WRITTEN` $\rightarrow$ `KANBAN_PENDING` $\rightarrow$ `COMPLETED` / `FAILED`
* **Transactional Outbox:** ทุกครั้งที่เขียน Obsidian Note สำเร็จ ต้องบันทึกสถานะและสร้าง Outbox Event ภายใน Atomic DB Transaction เดียวกัน เพื่อรับประกันว่างานจะไม่สูญหาย (At-Least-Once Delivery)
* **Outbox Worker:** Background Worker ดึง Event ไปส่งมอบ พร้อมระบบเช่าเวลา (Lease), กู้คืนงานค้าง (Lease Expiry Recovery), และนับจำนวน Retry สูงสุด

### 4.2 Idempotency & Fencing Token Lease
* **Deterministic Source Key:** คำนวณ Idempotency Key จาก canonical ticker, canonical period, transcript hash, และ prompt version เพื่อป้องกันการรันซ้ำ

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gnoMarkII/Trinity-Wealth-Engine](https://github.com/gnoMarkII/Trinity-Wealth-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
