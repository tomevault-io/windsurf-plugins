---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ZIP Production is a custom ERP system for a plastic zipper manufacturing company in Pakistan (client of BD Matrix agency), covering Production, Inventory, Gate Pass/Dispatch, Orders, Finance/Accounting, and HR. Governing product rules live in `.specify/memory/constitution.md` — read it before making architectural or RBAC decisions; it is the highest-authority spec document (data integrity/audit, RBAC, PKR currency conventions, mobile-first, offline resilience). Note: the constitution's original 5-role list (Super Admin/Finance/Production/Gate Operator/Viewer) has been superseded in code by 6 roles — see Roles below.

Feature spec, data model, and task breakdown for the current feature branch live in `specs/001-zipper-erp-system/` (`spec.md`, `data-model.md`, `plan.md`, `tasks.md`).

## Architecture

Two-service monorepo: `backend/` (Express 5 + TypeScript + Prisma/PostgreSQL) and `frontend/` (React 19 + Vite + TypeScript + Tailwind v4). Deployed via Docker (root `Dockerfile` builds the backend for Coolify; `docker-compose.yml` runs postgres + backend + frontend for local/staging).

### Backend request flow

`routes/*.routes.ts` → `middleware/auth.middleware.ts` (JWT from httpOnly cookie) → `middleware/rbac.middleware.ts` (role check against `config/roles.ts` `ROLE_PERMISSIONS`) → `controllers/*.controller.ts` → `services/*.service.ts` (business logic + Prisma) → `utils/serializer.ts` strips financial fields from the response for restricted roles before it goes out.

- **Roles** (`@prisma/client` `Role` enum): `SUPER_ADMIN`, `FINANCE_HEAD`, `PRODUCTION_HEAD`, `LOGISTICS_HEAD`, `MARKETING_HEAD`, `HR_HEAD`. Permission-to-role mapping is centralized in `backend/src/config/roles.ts` (`ROLE_PERMISSIONS`, `FINANCIAL_FIELD_RESTRICTED_ROLES`) — add new endpoints there, not ad hoc in controllers.
- **Financial field stripping**: `PRODUCTION_HEAD` and `MARKETING_HEAD` must never receive `*Paisa`/`*Display` monetary fields. This is enforced in `utils/serializer.ts` (`stripFinancialFields`), which controllers call explicitly on response payloads — it is not automatic, so new endpoints returning money fields must call it.
- **Money**: stored as integer paisa (bigint columns in Prisma), never floats. `utils/currency.ts` formats paisa → PKR display strings using South Asian lakh/crore grouping (`en-IN` locale), and parses shorthand (`2L`, `1.5Cr`) back to paisa. `app.ts` patches `BigInt.prototype.toJSON` globally so bigint paisa fields serialize correctly over JSON.
- **Cross-module transactions**: operations that touch multiple modules (e.g. Gate Pass → inventory deduction → client ledger entry; Production entry → raw material consumption → stock adjustment) MUST be wrapped in `prisma.$transaction(async (tx) => {...})` — see `gate-pass.service.ts`, `production.service.ts`, `order.service.ts` for the pattern. Do not split these into separate non-transactional calls.
- **Audit fields**: `created_by`/`updated_by`/timestamps are set server-side from `req.user.userId`, never trusted from client input.

### Frontend

- Route table lives in `frontend/src/router.tsx`: pages are lazy-loaded and wrapped in a `RoleGuard` that checks role against `utils/constants.ts` `ROLES`. When adding a page, add both the lazy import and the `RoleGuard` role list — the backend `ROLE_PERMISSIONS` and this route guard must be kept in sync manually.
- `services/*.api.ts` — one file per module, thin wrappers around the shared `services/api.ts` axios instance.
- `services/api.ts` has two response interceptors: one redirects to `/login` on 401, and one (`offlineQueue.ts` via `mutationQueue`) queues failed write mutations (POST/PUT/PATCH/DELETE on network error) in IndexedDB and returns a synthetic 202 so the UI doesn't break — this is the offline-resilience mechanism from the constitution. Mutations replay when connectivity resumes; replay requests set `_skipOfflineQueue` to avoid re-queuing.
- `hooks/useSmartQuery.ts` wraps TanStack Query with project conventions; prefer it over calling `useQuery` directly in new pages.
- Auth/role state: `context/AuthContext.tsx`, `hooks/useAuth.ts`, `hooks/useRole.ts`.

### Database

`backend/prisma/schema.prisma` is the single source of truth for ~40 models spanning Production (ZipperVariant, Recipe, ProductionEntry, GrainType), Inventory (RawMaterialStock, FinishedGoodsStock, RawMaterialBatch), Gate Pass/Dispatch (GatePass, SalesReturn), Orders/Clients (Order, Client, ClientRate), Accounting (Account, JournalEntry, JournalEntryLine, Voucher — double-entry, debits must equal credits), HR (Worker, WorkerSalaryRate, PayrollRecord, WorkerAdvance), and AuditLog/Notification/SystemSetting. `backend/prisma/seed.ts` is idempotent and auto-runs on container startup (see Dockerfile `CMD`), seeding production master data (grain types, packaging, plant, machines, variants, electricity rates, overheads) — keep it safe to re-run.

## Commands

Run from `backend/` or `frontend/` respectively (no root-level package.json/workspace tooling).

### Backend (`backend/`)
```
npm run dev             # ts-node-dev, auto-restart

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ashhad1200/ZIP-Production](https://github.com/Ashhad1200/ZIP-Production) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
