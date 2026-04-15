---
trigger: always_on
description: Critical context and instructions for AI agents working in this repository.
---


Critical context and instructions for AI agents working in this repository.

## Project Overview

**StratCol Onboard Control Tower** is a high-stakes, event-driven onboarding automation platform. It manages the full lifecycle of applicant onboarding—from lead capture to final approval—without middleware.

- **Architecture:** 6-stage pipeline orchestrated by [Inngest](https://www.inngest.com/).
- **Framework:** [Next.js](https://nextjs.org/) (App Router, Turbopack).
- **Runtime & Tooling:** [Bun](https://bun.sh/) is the mandatory package manager and runtime.
- **Database:** [Turso](https://turso.tech/) (LibSQL/SQLite) with [Drizzle ORM](https://orm.drizzle.team/).
- **Auth:** [Clerk](https://clerk.com/).
- **AI Integration:** Multiple Gemini-powered agents (`lib/services/agents/`) for risk, sanctions, and verification.
- **Telemetry:** Extensive PostHog instrumentation with "Perimeter Validation" for security and compliance.

## Building and Running

| Command | Purpose |
|---------|---------|
| `bun dev` | Start Next.js dev server (Turbopack) |
| `bun run dev:all` | Start Next.js + Inngest dev server |
| `bun run build` | Production build |
| `bun run lint` | Run Biome linting |
| `bun run lint:fix` | Biome lint with auto-fix |
| `bun run db:generate` | Generate Drizzle migrations |
| `bun run db:migrate` | Apply migrations to dev DB |
| `bun run db:reset` | Full DB reset (Dev + Test) |
| `bun run test:e2e` | Run Playwright E2E tests |
| `bun run dev:browser-flow` | Start dedicated browser-flow test stack (Port 3100) |

## Development Conventions

### Core Mandates
- **Bun Only:** Use `bun` for all operations. Never use `npm`, `yarn`, or `pnpm`.
- **Strict Typing:** No `any` types. Biome enforces strict correctness.
- **No `console.log`:** Use `console.assert`, `console.warn`, or `console.error`.
- **Server Actions:** Must authenticate internally via `requireAuth()`.
- **Inngest Reliability:** Steps must be replay-safe; deterministic logic only.

### Workflow & Testing
- **Browser-flow Verification:** Required for any changes touching the dashboard or dev server. Use `bun run dev:browser-flow` on port 3100.
- **Database Rules:** Primary data is **SQLite/LibSQL**. Use `json_extract` for JSON fields.
- **Contractual APIs:** Breaking changes to applicant intake (`app/api/applicants`) require explicit approval.
- **Testing Isolation:** `E2E_USE_TEST_DB=1` switches the app to the test database.

### AI Agent Architecture
AI agents are located in `lib/services/agents/` and use Google Gemini:
- **Validation Agent:** Document and data validation.
- **Risk Agent:** Overall risk level assessment (`green`, `amber`, `red`).
- **Sanctions Agent:** Cross-referencing against OpenSanctions and Firecrawl.
- **Reporter Agent:** Generating final compliance reports.

## Key Directories

| Path | Description |
|------|-------------|
| `app/` | Next.js routes; `(authenticated)/` is the dashboard. |
| `inngest/` | Workflow definitions and stage handlers. |
| `lib/services/` | Business logic for risk, sanctions, FICA, etc. |
| `lib/ai/` | GoogleGenAI wrappers and model configurations. |
| `db/schema.ts` | The single-file source of truth for the Drizzle schema. |
| `docs/` | Comprehensive Mintlify documentation. |
| `tests/` | Playwright E2E and staged browser-flow scripts. |

## Documentation (Mintlify)
The documentation is located in `docs/`. To run locally:
1. `bun i -g mint`
2. `cd docs && mint dev`


Learned User Preferences

- Browser-flow and similar UI verification with agent-browser is required before claiming a feature or fix is done—not optional when the work touches the dashboard or dev server behavior.
- Do not run `git commit` or `git push` unless the user explicitly asks.
- For narrow test-fix or debugging tasks, do not change production application code without explicit permission; prefer tests, scripts, env, or docs unless the user expands scope.
- When editing applicant intake (`app/api/applicants`, applicant forms, validations), treat registration and mandate-related API behavior as contractual unless the user approves a breaking change.
- When delegating to long-running subagents (for example deep Inngest or audit passes), wait for them to finish before summarizing, closing the task, or assuming results.
- For `/compound-mintlify` (and the compound-mintlify agent workflow), publish finalized `docs/issues/` markdown to the StratCol AI ClickUp wiki **without** asking “Sync to ClickUp?” when the **`user-clickup`** MCP is available; if MCP is unavailable or the wiki step fails, report that explicitly instead of implying success.

## Learned Workspace Facts

- Cursor agent transcripts for this repo may live under the multi-root workspace project `Users-jakwakwa-Documents-code-workspaces-Scol-Control-Tower-code-workspace` rather than under `Repos/.../Scol-Control-Tower` alone.
- `getDatabaseClient()` uses `TEST_DATABASE_URL` only when `E2E_USE_TEST_DB` equals `1`; Playwright’s web server already sets this for E2E runs.
- Primary app data is **SQLite/LibSQL** (Turso)—**not Postgres**; plans and raw SQL should use SQLite/JSON1 semantics (e.g. `json_extract`), not Postgres-only dialects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jakwakwa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
