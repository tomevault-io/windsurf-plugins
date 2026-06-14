---
trigger: always_on
description: This file defines mandatory development rules for AI Agents in this project. Unless the user explicitly requests a deviation, these rules must be followed.
---

# AGENTS.md

This file defines mandatory development rules for AI Agents in this project. Unless the user explicitly requests a deviation, these rules must be followed.

## 1. Basic Principles

- Scope: the repository root and all subdirectories
- Priority: explicit user instructions > this file > default implementation habits
- If these rules conflict with the user's request: follow the user's request first, and note the deviation in the change summary

## 2. Fixed Technology Stack

- Backend: `Golang` + `Gin` + `GORM` + `github.com/mlogclub/simple`
- Database: must be compatible with both `SQLite` and `MySQL`
- Frontend: `Next.js(App Router)` + `React` + `shadcn/ui` + `Tailwind CSS`
- Frontend package manager: `pnpm`

## 3. Directory Conventions

```text
.
├── cmd/
│   ├── server/
│   ├── migration/
│   └── generator/
├── internal/
│   ├── bootstrap/
│   ├── builders/
│   ├── handlers/
│   │   ├── api/
│   │   ├── dashboard/
│   │   └── third/
│   ├── middleware/
│   ├── migration/
│   ├── models/
│   ├── repositories/
│   ├── services/
│   └── pkg/
│       ├── config/
│       ├── dto/
│       ├── enums/
│       ├── errorsx/
│       ├── httpx/
│       ├── logx/
│       └── utils/
├── web/
└── docs/
```

## 4. Backend Layering

The backend must follow one-way dependencies: `models -> repositories -> services -> handlers`

- `models`: only define entities and table mappings
- `repositories`: only encapsulate data access
- `services`: handle business rules, transaction orchestration, and aggregation logic
- `handlers`: only parse parameters, check permissions, call services, and wrap responses

Forbidden:

- Handlers directly calling repositories
- Returning GORM models directly to the frontend
- Writing business orchestration in models or repositories

## 4.1 Full Layer Flow (`models -> repositories -> services -> handlers -> builders`)

This section is an executable refinement of the layering rules: each layer must do only what belongs to that layer. Data should flow around DTOs, GORM details should be concentrated in repositories, transaction boundaries should be concentrated in services, and response assembly should be concentrated in builders/handlers.

### 4.1.1 Dependency Direction (Required)

Only the following one-way dependencies are allowed:

- `models` -> must not depend on any business layer
- `repositories` -> may depend on `models` and base libraries (`gorm`/`simple/sqls`)
- `services` -> may depend on `repositories`, `models`, and `enums/errorsx/utils`; responsible for transactions and business orchestration
- `builders` -> may depend on `models` and `dto/response`; if necessary, may depend on a small number of `services` to supplement display fields, but aggregation in the service layer is preferred
- `handlers` -> may depend on `services`, `builders`, `pkg/dto/request`, `pkg/httpx/params`, and `pkg/httpx` response wrappers

Reverse dependencies are forbidden:

- `repositories` must not depend on `services/handlers/builders`
- `models` must not depend on `repositories/services/handlers/builders`
- `handlers` must not depend on `repositories` (they must go through services)

### 4.1.2 Data Shape and Flow (Recommended Standard)

A typical CRUD/business action data flow:

1. The **handler** reads parameters (`query/body/form/path`), performs permission checks, and calls the **service**
2. The **service** executes business rules (validation, idempotency, state machines, aggregation), starts a transaction when needed, and calls the **repository**
3. The **repository** only performs data reads/writes (`CRUD + queries`) and returns `models` or necessary aggregate structures
4. **builders** map `models`/aggregate results into `response DTO`
5. The **handler** returns `httpx.WriteJSON(...)`

Strong constraints:

- **Handler inputs use request DTOs**
- **Handler outputs use response DTOs**
- **Models must not be returned directly to the frontend**

### 4.1.3 Per-Layer Allow/Forbid Checklist

#### models (Entity Layer)

- **Allowed**
  - Field definitions, table names, index/constraint tags, associations (GORM tags)
  - Lightweight constants/enum field types (prefer `internal/pkg/enums`)
- **Forbidden**
  - Business methods (for example, rule checks such as `CanDispatch()` belong in services)
  - DB access, transactions, complex calculations

#### repositories (Data Access Layer)

- **Allowed**
  - CRUD: `Get/Take/Find/FindOne/FindPageBy.../Create/Update/Updates/UpdateColumn/Delete`
  - Reusable query-related methods: `FindByUserID`, `CountByStatus`, `FindActiveBy...`
  - Anything that is a data-access detail belongs here (SQL conditions, sorting, pagination, locks)
- **Forbidden**
  - Business orchestration (cross-table workflows, state transitions, event publishing, etc.)
  - Permission checks or login-state checks
  - Directly assembling response DTOs (DTO mapping belongs in builders/handlers)

Repository best practices:

- **Prefer unified primary-key read/write methods**: `Get/Updates/Delete`, avoiding repeated `id = ?` logic in services
- **Prefer query conditions through `sqls.Cnd` / `sqls.NewCnd()`**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huabeitech/agent-desk](https://github.com/huabeitech/agent-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
