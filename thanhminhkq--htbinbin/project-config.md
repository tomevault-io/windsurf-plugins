---
trigger: always_on
description: 1. Preserve security and permissions.
---

# Bin Bin Hotel Management System

# AI Engineering Operating System

## Priority Order

1. Preserve security and permissions.
2. Preserve reservation and inventory integrity.
3. Preserve existing business logic.
4. Preserve execution flow stability.
5. Preserve UI/UX consistency.
6. Improve code quality only when directly related.

## Mission

Bin Bin Hotel Management System handles:

- PMS reservations.
- OTA booking ingestion.
- Room inventory and pricing.
- CRM workflows.
- Operational hotel management flows.
- HR attendance, tasks, and lost-and-found workflows.

## Technology Stack

### Backend

- Python.
- FastAPI.
- Starlette.
- Pydantic.
- SQLAlchemy.

### Database

- PostgreSQL.
- Supabase.
- Alembic migrations.

### Frontend

- Jinja templates.
- TailwindCSS.
- Vanilla JavaScript.
- Static assets in `src/frontend/static`.

### Jobs & Integrations

- APScheduler.
- Gmail APIs.
- Google APIs.
- OTA workflows.
- Redis/cache support.

### Testing

- pytest.
- Tests under `tests/`.

## Architecture Rules

### Backend

Keep:

- API/session/request handling in `src/backend/api/` and `src/backend/main.py`.
- Business workflows in `src/backend/services/`.
- Database models/utilities in `src/backend/db/`.
- Migrations in `alembic/`.
- Schemas in `src/backend/schemas/`.

Domain grouping (từ 2026-07):

- `src/backend/api/` gom theo domain: `hr/`, `operations/`, `ota/`, `system/`, `pms/`, `inventory/` — file route mới đặt vào đúng package domain.
- `src/backend/db/models/` là package tách theo domain; import model qua `from src.backend.db.models import X` (re-export từ `__init__.py`).
- `src/backend/services/identity/` (parser giấy tờ), `src/backend/services/vn_address/` (địa chỉ VN) — logic parse KHÔNG đặt trong `src/backend/api/`.
- Một instance Jinja2Templates duy nhất: `src/backend/core/templates.py`. Không tự tạo instance mới trong route module.
- Đường dẫn data/log lấy từ `src.backend.core.config` (`STATIC_DATA_DIR`, `LOG_DIR`, `APP_DIR`, `FRONTEND_DIR`), không tính từ `__file__`.
- Route snapshot test (`tests/test_app_smoke.py`) phải pass sau mọi thay đổi route; đổi route có chủ đích thì regenerate baseline (xem `.claude/commands/test.md`).

### Frontend

Keep:

- Presentation logic in `src/frontend/templates/` and `src/frontend/static/`.
- Không viết `<style>`/`<script>` logic trong template; CSS ra `static/css/{domain}/`, JS ra `static/js/{domain}/`. Biến từ server truyền qua `window.PAGE_DATA`.
- Keyframes dùng chung khai báo ở `static/css/shared/components.css` với prefix `bb-`; không đặt trùng tên keyframes của Tailwind (`fadeIn`, `pulse`, `ping`, `fade-in-up`).
- Chi tiết đầy đủ: `docs/frontend-conventions.md`.

Preserve:

- Desktop layout integrity.
- Mobile responsiveness.
- Touch-friendly interactions.
- Consistent spacing and hierarchy.
- Existing interaction patterns.

Avoid:

- Unnecessary animations.
- Layout instability.
- Client-side business rules when server state should decide.

## Critical Invariants

These areas are high-risk and require extra caution:

- Reservation state transitions.
- Inventory state transitions.
- Check-in/check-out flows.
- Branch/user visibility rules.
- Auth/session boundaries.
- OTA parsing and matching logic.
- Pricing calculations.
- Cross-module utilities.
- Execution-critical functions.

Do not modify these blindly. Always inspect before changing behavior:

- Execution flows.
- Affected callers.
- Downstream dependencies.
- Related services.
- Permissions/session scope.

Never expose secrets in code. Keep secrets in environment variables. Never bypass permission checks for convenience.

# Engineering Rules

## Decision Rules

Before implementing:

- State assumptions explicitly when uncertainty exists.
- If multiple interpretations exist, ask or present options.
- Prefer the simplest working solution.
- Push against unnecessary complexity.
- Do not silently introduce architectural changes.
- If scope is unclear, stop and clarify first.

## Simplicity Rules

- Prefer the smallest working solution.
- Avoid speculative abstractions.
- Avoid abstractions for single-use logic.
- Reuse existing patterns before introducing new architecture.
- Avoid unnecessary dependencies.
- Avoid premature optimization.
- Prefer readability over cleverness.
- If a solution feels overly generic, simplify it.
- Do not create reusable abstractions until duplication or repetition is proven.

## Surgical Change Rules

- Touch only files directly related to the task.
- Do not refactor unrelated code.
- Do not rewrite stable working code unnecessarily.
- Preserve existing naming and structure.
- Match existing project style and patterns.
- Remove only artifacts introduced by your own changes.
- Keep change directly tied to the requested task.

If unrelated issues are discovered:

- Mention them separately.
- Do not silently fix them.

## Scope Discipline

Do not expand scope without explicit approval.

If additional improvements are discovered:

- Mention them separately.
- Do not implement automatically.

Avoid:

- Opportunistic rewrites.
- Cleanup-only edits.
- Unrelated refactors during feature work.

## Stability Rules

- Prefer stable working code over idealized refactors.
- Minimize regression risk over code elegance.
- Avoid broad rewrites unless explicitly requested.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThanhMinhKQ/htbinbin](https://github.com/ThanhMinhKQ/htbinbin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
