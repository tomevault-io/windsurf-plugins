---
trigger: always_on
description: > Binding context for every AI coding agent (Claude Code, Cursor, Cline, etc.) working in this repo. Read this **before** writing code.
---

# CLAUDE.md — Suitest coding rules

> Binding context for every AI coding agent (Claude Code, Cursor, Cline, etc.) working in this repo. Read this **before** writing code.
>
> After the OSS pivot (2026-05-26), Suitest = **Python/FastAPI backend** + **Vite/React frontend** + **MCP-native plugin layer** + **capability tiering**.

---

## 1. What is Suitest

**MCP-native testing platform. Manual TCM, deterministic runs, autonomous AI when configured. Your stack, your LLM, your data.**

A self-hostable OSS platform that combines:

- **Manual TCM** — traditional test case editor (steps, expected, assertions, traceability)
- **Deterministic runner** — step execution via pluggable MCP servers (Playwright, API HTTP, Postgres, GraphQL, gRPC, Mongo, Kubernetes, custom)
- **AI generation (optional)** — when the user brings their own LLM key, agents generate from PRD / OpenAPI / URL / MCP discovery
- **AI diagnosis (optional)** — auto-categorize defects (FLAKE / REGRESSION / ENVIRONMENT / TEST_BUG) when an LLM is available
- **Capability-tiered** — features grow automatically from ZERO → LOCAL → CLOUD based on user configuration

Positioning: a replacement for TestRail + Playwright (ZERO tier) that also goes beyond TestSprite (CLOUD/LOCAL tier) without vendor lock-in.

Visual reference: [`docs/UI_SPEC.md`](./docs/UI_SPEC.md).

---

## 2. Working rules

### 2.1 Always do this first

**`docs/ROADMAP.md` is the single entry point.** To continue any feature, start there — pick the next acceptance criterion that is not yet `[x]` in the active milestone. The ROADMAP determines order and scope; do not work from memory or from another doc first.

Other spec docs are **conditional references**, opened ONLY when the ROADMAP item you are working on needs their detail:

| Open this doc | Only when the ROADMAP item… |
|--------------|---------------------------|
| `docs/PRODUCT.md` | needs feature behavior/persona context |
| `docs/UI_SPEC.md` | touches the frontend (components are already specified) |
| `docs/API.md` | adds/changes an endpoint |
| `docs/DATA_MODEL.md` | touches the schema — do not invent columns without a spec update + Alembic migration |
| `docs/CAPABILITY_TIERS.md` | is LLM-dependent — you must know the tier gating |
| `docs/MCP_PLUGINS.md` | touches the runner / MCP routing |
| `docs/AUTONOMY.md` | is an agentic action with side effects |

Every doc has a build-status banner at the top (built vs spec M2–M4) — read it before trusting the contents. If the ROADMAP and a spec conflict, **the ROADMAP wins**; update the spec in the same PR.

### 2.2 Don't

- Don't add new dependencies without updating `docs/ARCHITECTURE.md`
- Don't create persistent "demo data" — always use a Python seed script
- Don't hardcode credentials, API keys, or production URLs — use env vars
- Don't write barrel files (`__init__.py` re-exporting everything / `index.ts` re-exports) — import directly
- Don't use `Any` in Python (mypy strict) — use specific types, `TypedDict`, or `Protocol`
- Don't use `as any` in TypeScript — use narrowing / `unknown` + a Zod validator
- Don't call LLM SDKs directly from API routes — always go through `packages/agent` via LiteLLM
- Don't call MCP servers directly from API routes — always go through `packages/mcp/client`
- Don't skip **capability gating** for AI features — an LLM feature without `require_tier(...)` is a BUG
- Don't store any secret in plaintext — always AES-GCM via `packages/core/crypto`
- Don't skip the audit log for mutations — every write operation logs via the `audit_log` table

### 2.3 Must

- **Python 3.12** typed (mypy strict, `disallow_untyped_defs = true`)
- **Pydantic v2** schemas for all API input/output + DTOs
- **SQLAlchemy 2 async** + **Alembic** for all DB access (no raw SQL except performance-critical, with a `# perf: raw SQL` comment)
- **FastAPI** + dependency injection (no globals; every service injected via `Depends`)
- **Ruff** + **Black** + isort configured (one tool: ruff format)
- **pytest async** for testing (`pytest-asyncio` strict mode)
- **FE TypeScript strict** mode; Zod schemas validating API I/O on the client
- All **AI calls** go through `packages/agent` (LiteLLM router)
- All **MCP calls** go through `packages/mcp/client` (registry + pool)
- All **DB access** goes through the repository pattern (`packages/db/repositories/*.py`)
- **AES-GCM** for stored secrets via `packages/core/crypto`
- **Audit log** every mutation (`packages/db/audit.py`)
- Every new endpoint must declare its tier requirement via `Depends(require_tier(...))`
- Every LLM-dependent UI feature must be wrapped in `<Gated feature="...">`

---

## 3. Code conventions

### 3.1 Naming

- Python files: `snake_case.py`
- TS module files: `kebab-case.ts`
- React components: `PascalCase.tsx`
- DB tables: `snake_case`, plural (`test_cases`, `test_runs`, `mcp_providers`)
- API routes: `kebab-case` plural (`/api/v1/test-cases`, `/api/v1/mcp/providers`)
- Env vars: `SCREAMING_SNAKE_CASE`, prefix `SUITEST_*` (e.g. `SUITEST_DATABASE_URL`, `SUITEST_REDIS_URL`). The LLM is **not** configured via env — it is configured per-workspace from the web UI.
- Python classes: `PascalCase`
- Python functions/vars: `snake_case`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suiflex/suitest](https://github.com/suiflex/suitest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
