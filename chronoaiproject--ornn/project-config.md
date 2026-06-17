---
trigger: always_on
description: **Ornn is an agent-facing skill-lifecycle API, not a human marketplace.**
---

# CLAUDE.md — chrono-ornn

## Product Positioning

**Ornn is an agent-facing skill-lifecycle API, not a human marketplace.**

The primary customer is the AI agent developer / agentic-system builder. Agents call Ornn directly — over HTTP or MCP — to manage their own skill lifecycle: search → pull → install → execute → build → upload → share. Closest analog: **npm registry + npm CLI fused, model-agnostic** (works for Claude / GPT / Gemini / custom — not locked to one model runtime).

Implications when proposing or building features:

- Lead with the **agent-API contract** (REST / MCP ergonomics, stable schemas, model-agnostic guarantees) before any human-UX angle.
- `ornn-web` is a *secondary* surface for skill owners and platform admins — it is not the primary product. UI features that don't translate into agent-API value are deprioritized.
- Avoid feature framing that drifts toward "another skill marketplace" (social ranking, browse-style discovery, recommendation feeds, leaderboards) unless we deliberately decide to. When a feature looks marketplace-shaped, surface that tension before building.

## Tech Stack

TypeScript, Bun workspace monorepo

- **Runtime:** Bun (backend + tests), Vite (frontend dev/build)
- **Backend:** Hono
- **Frontend:** React 19, Zustand, TanStack Query, Tailwind CSS 4, Framer Motion, React Router 7
- **Database:** MongoDB 7
- **Validation:** Zod
- **Logging:** Pino
- **Testing:** Bun test (backend); Vitest + Testing Library + jsdom (frontend + TS SDK); pytest + respx (Python SDK). All run in CI — bun packages via `bun run test`, Python via a dedicated `python-sdk-test` job.

**Packages:**

| Package | Path | Description |
|---------|------|-------------|
| `ornn-api` | `ornn-api/` | Backend API (Bun + Hono + MongoDB) |
| `ornn-web` | `ornn-web/` | React SPA (Vite + React 19 + Zustand + TanStack Query) |
| `@chronoai/ornn-sdk` | `sdk/typescript/` | TypeScript client for `/api/v1/*` |
| `ornn-sdk` (Python) | `sdk/python/` | Python client for `/api/v1/*` (httpx) — separate release cadence |

## Architecture

- Two packages: `ornn-api` (backend) and `ornn-web` (web UI).
- All configurable values MUST be read from environment variables. Zero hardcoded config.
- For project domain knowledge (external services, skill format, etc.), see `docs/ARCHITECTURE.md`.

## Code Standards

- TypeScript + Bun. Follow TypeScript and Bun conventions.
- Use `Result` patterns and Zod validation. No bare `try/catch` in routes — use error middleware.
- Keep code simple. Fewer lines > more abstractions.
- All code MUST include sufficient logging (Pino). `info` for lifecycle events, `debug` for detailed flow, `error` for failures with context.
- Logs MUST NOT contain plaintext secrets. Mask or redact sensitive values.
- No hardcoded secrets, credentials, API keys, tokens in code — ever.
- Tests: `bun run test` runs both (backend via Bun, frontend via Vitest).
- Unit tests colocated with source files. Integration tests in `tests/` directory.
- Always use Docker to run and test locally. Do not run services directly with `bun run dev`. CI builds both `ornn-api` and `ornn-web` images on every PR (`docker-build` job) so Dockerfile breakage surfaces immediately.

## Branching Strategy

- **`main`** — Production release branch. Protected: no direct push, no force push, PRs only from `develop`.
- **`develop`** — Default branch and active development branch. Contains the latest CI-passing code. Protected: no direct push, no force push, PRs from any feature branch.
- **Workflow:** `feature/xxx` → PR → `develop` → PR → `main`
- PR merge auto-deletes the source branch (protected branches excluded).
- **New work MUST branch from the latest `origin/develop`.** Every feature, bug fix, or any kind of change must start from a freshly fetched `develop` — either a new branch (`git fetch && git checkout develop && git pull && git checkout -b <name>`) or a new worktree created against `origin/develop`. Never branch off a stale local `develop` or another feature branch.

## Commit Standards

Every code change MUST respect commit size. **Multiple small, self-contained commits are always better than one big combined commit, no exceptions.**

Rules:

1. **Each commit is self-contained.** A reviewer reading just that commit's diff + message can understand what changed and why, without context from sibling commits.
2. **Each commit is small.** One logical change per commit. Schema migration is one commit. New endpoint is another. Frontend drawer is another. Old page deletion is another. Tests can ride with the code they test, but unrelated test additions get their own commit.
3. **Each commit's message is detailed.** Subject line states the change in one short sentence (≤ 72 chars). Body explains the *why*, the constraints, and any non-obvious decisions. Reference the issue when relevant (`#270`).
4. **Order matters.** Stack commits in dependency order so the tree compiles + tests pass at every commit (or as close as is practical). A reviewer should be able to `git checkout` any intermediate commit without a broken state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChronoAIProject/Ornn](https://github.com/ChronoAIProject/Ornn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
