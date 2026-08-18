---
trigger: always_on
description: Canonical agent instructions for this repo. **`CLAUDE.md`** and **`.cursor/rules/agents.mdc`** point here; read the relevant section before changing that area.
---

# AGENTS.md

Canonical agent instructions for this repo. **`CLAUDE.md`** and **`.cursor/rules/agents.mdc`** point here; read the relevant section before changing that area.

## Quick commands

```bash
npm run build          # backend + frontend
```

Run the stack: see root `README.md` (`docker compose up -d`).

## Pull requests

**Branch names** (enforced in CI): `<type>/<short-description>` in lowercase kebab-case.

Types: `feat` · `fix` · `test` · `task` · `chore` · `docs` · `refactor` — see **`.github/BRANCH_NAMING.md`**.

Examples: `feat/custom-tools-ui`, `fix/contact-pagination`, `test/custom-tools-e2e`.

When creating a PR, fill out **`.github/pull_request_template.md`**: what & why, user impact, migrations, tests, README, and how to verify.

## Stack

| Layer | Technology |
|-------|------------|
| Frontend | Vite, React 19, React Router, Tailwind 4, shadcn / Base UI |
| Backend | Hono (Node), Drizzle ORM, PostgreSQL |
| Auth | JWT issued by backend (`backend/src/routes/auth.ts`, middleware) |
| Files | S3-compatible storage (MinIO / R2 / AWS) via `backend/src/lib/storage.ts` |
| Jobs | pg-boss, SMTP (nodemailer), OpenRouter (see `README.md`) |

| Package | Role |
|---------|------|
| **`frontend/`** | Vite + React 19 SPA, React Router, Tailwind 4, shadcn/Base UI — talks to backend over HTTP (`frontend/src/lib/api.ts`) |
| **`backend/`** | Hono API on Node, Drizzle ORM → PostgreSQL, JWT auth, S3-compatible storage |
| **`whatsapp/`** | WhatsApp session manager (separate service) |
| **`database/`** | Drizzle SQL migrations (`database/migrations/`) and ops scripts (`database/scripts/`) |

**Out of scope for new work:** Next.js, Supabase JS client, server components, server actions.

**Not in use:** Next.js App Router, Supabase client in app code, or a root-level `landing/` app (removed from the tree).

---

# Production rules

## Design

- Customer-facing UI follows **`DESIGN.md`** (see [Design system](#design-system-source-of-truth)). Tokens live in **`frontend/src/globals.css`**.

## Architecture (strict)

### 1. Repository pattern (backend only)

All **database** access lives in **`backend/src/repositories/`**.

- **Frontend** must never query Postgres or call Drizzle.
- **Services** (`backend/src/services/`) must not use Drizzle directly — go through repositories.
- Repositories use **`backend/src/db/index.ts`** (Drizzle) and schema under **`backend/src/db/schema/`**.

### 2. Repository logging + error handling

Every repository function must:

- Use try/catch
- Log success, expected failure, and unexpected failure

#### Log format

**Success:** `[SettingsRepository/updateProfile] Success: userId=123`

**Expected failure:** `[SettingsRepository/updateProfile] Failed query: <message>`

**Unexpected (mandatory):** `[SettingsRepository/updateProfile] Unexpected error: <message>`

### 3. Database migrations

- Schema is defined in **`backend/src/db/schema/`**.
- **Drizzle** migrations live in **`database/migrations/`** (generate with `drizzle-kit` from `backend/drizzle.config.ts`).
- Add a **new** migration file per change; do not edit applied migration SQL in place.
- **`database/scripts/`** holds optional ops SQL (e.g. owner setup); schema changes go through Drizzle only.
- No manual production schema changes outside migrations.

### 4. Frontend UI layout

Feature UI belongs under:

```
frontend/src/pages/<feature>/components/
```

Examples: `pages/dashboard/components/`, `pages/settings/components/`.

Rules:

- UI stays **presentational** — no business rules buried in JSX.
- UI must **not** import repositories.
- Data and mutations go through **hooks** (`frontend/src/hooks/`) and **`frontend/src/lib/api.ts`**.

### 5. Component size

- Max **150 lines** per component; split when responsibilities diverge.

### 6. Data flow

```
Browser → frontend hook → lib/api.ts → Hono route → service (optional) → repository → Postgres
```

Forbidden:

- Frontend → database
- Route handler → inline SQL (use repositories)

### 7. React (SPA)

- Functional components only; no side effects in render.
- Hooks: `use*` prefix, single responsibility, full dependency arrays when using `useEffect`.
- Prefer hooks + `api.ts` for server state; avoid `useEffect` for initial fetch when a dedicated hook already exists.
- State priority: local state → derived state → context.

### 8. Backend API

- Routes in **`backend/src/routes/`**; mount in **`backend/src/index.ts`**.
- Validate input (e.g. Zod) at the route boundary.
- Enforce auth via **`backend/src/middleware/auth.ts`** and workspace scope via **`X-Workspace-Id`** where applicable.
- Never expose DB credentials or S3 secrets to the frontend.
- **No unused return payloads** — Do not include fields on API responses, hook return values, or `RunAgentResult`-style objects unless a real caller uses them. Prefer computing/persisting side effects inside the function (e.g. operator reasoning → DB) over echoing them back. When changing a contract, drop dead fields in the same change.

### 9. Code style

- TypeScript strict; avoid `any`.
- **Frontend** imports: `@/*` → `frontend/src/*`.
- **Backend** imports: relative paths under `backend/src/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devennn/senqo](https://github.com/devennn/senqo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
