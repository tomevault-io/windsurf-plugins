---
trigger: always_on
description: - Keep Alt's Compose-first stack healthy while accelerating developer workflows.
---

# AGENTS Playbook

## Mission & Scope
- Keep Alt's Compose-first stack healthy while accelerating developer workflows.
- Document the guardrails for AI coding agents and human collaborators working inside this monorepo.
- Treat Kubernetes assets in `stopped-using-k8s/` as historical only—do not modify them unless specifically asked.

## Operating Constraints (October 2025)
- Filesystem access: `workspace-write`; touch only the workspace and declared writable roots.
- Network access: restricted. Never run commands that require outbound traffic without approval.
- Approvals: `on-request`. Ask for escalation only when essential; otherwise work within the sandbox.
- Default tools: prefer `rg` for search and `sed -n` for focused reads; keep command output under 250 lines.
- Patching: use `apply_patch` for code and doc edits. Group related changes, avoid unrelated refactors.
- Planning: maintain a live plan with `update_plan`, exactly one step `in_progress` at a time.
- TDD First: every service expects Red → Green → Refactor before shipping implementation work.

## Baseline Workflow for Agents
1. **Confirm context** – Read this playbook plus relevant service `CLAUDE.md` files before touching code.
2. **Explore lightly** – Skim directories with `rg --files` and targeted `sed -n` reads. Avoid dumping large files.
3. **Plan** – Announce a concise multi-step plan via `update_plan` and keep it current.
4. **Implement surgically** – Edit only what the task requires; use tight diff scopes with `apply_patch`.
5. **Verify** – Run the smallest meaningful test or build command for the affected area.
6. **Communicate** – Summarize changes, list tests, surface risks, and suggest next actions.

## Repository Map
- `alt-frontend-sv/` – SvelteKit 2 + Svelte 5 Runes client (TailwindCSS v4, Vitest, Playwright).
- `alt-backend/app/` – Go 1.26 HTTP API in Clean Architecture layers.
- `alt-backend/sidecar-proxy/` – Go egress proxy enforcing outbound policy.
- `pre-processor/app/` – Go feed and summarization worker with circuit breakers.
- `pre-processor-sidecar/app/` – Go scheduler for Inoreader ingestion (CronJob/deployment).
- `news-creator/app/` – FastAPI LLM service using Ollama via Clean Architecture.
- `tag-generator/app/` – FastAPI + Python 3.14 tag pipeline with ML components.
- `search-indexer/app/` – Go Meilisearch indexer and search API.
- `auth-hub/` – Go IAP service bridging Nginx and Ory Kratos.
- `auth-token-manager/` – Deno OAuth2 token refresher for Inoreader.
- `knowledge-sovereign/` – Go durable knowledge state owner (Trail / event log / projectors).
- `rask-log-forwarder/` & `rask-log-aggregator/` – Rust log pipeline (forwarder + ClickHouse aggregator).
- Support assets: `compose/compose.yaml` (include stacks), `Makefile`, `scripts/`, `docker/`, `db/`, `.github/`, root `tests/`.

## Core Tooling & Commands
- **Stack orchestration**
  - Prefer: `docker compose -f compose/compose.yaml -p alt up -d` (see root `CLAUDE.md`).
  - `make up` – Copies `.env.template` → `.env` if needed, builds images, starts Docker Compose.
  - `make down` / `make down-volumes` – Stop stack (keep vs. drop volumes).
  - Stacks are Compose **`include:`** files under `compose/` (`ai.yaml`, `logging.yaml`, `sovereign.yaml`, …), not root-level `--profile` flags.
- **Frontend (SvelteKit / bun)**
  - Dev server: `cd alt-frontend-sv && bun dev`
  - Build: `cd alt-frontend-sv && bun run build`
  - Tests: `cd alt-frontend-sv && bun run test` (unit), `bun run test:e2e` (requires stack), coverage via `test:coverage`
  - Quality gates: `cd alt-frontend-sv && bun run format`, `bun run lint`, `bun run check`
- **Backend & Go services**
  - Go tests: `cd <service> && go test ./...` (add `-race -cover` when appropriate)
  - Formatting: `gofmt`, linting via `go vet`
  - Mock generation: `make generate-mocks`
- **Python services (news-creator, tag-generator)**
  - Tests: `SERVICE_SECRET=test-secret pytest` (news-creator), `uv run pytest` (tag-generator)
  - Type/lint: `uv run mypy`, `uv run ruff check`, `uv run ruff format`
- **Rust services (rask-*)**
  - Unit/integration: `cargo test`
  - Benchmarks: `cargo bench` (when explicitly required)
- **Deno (auth-token-manager)**
  - Tests: `deno test`
- **Health checks**
  - Frontend (via nginx): `curl http://localhost/health`
  - Backend: `curl http://localhost:9000/v1/health`
  - BFF: `curl http://localhost:9250/health`
  - Meilisearch: `curl http://localhost:7700/health`
  - Auth Hub: `curl http://localhost:8888/health`

## Language Playbooks
- **Go 1.26** – Enforce Clean Architecture boundaries, use `log/slog`, wrap errors with context, propagate `context.Context`, throttle external calls (≥5 s between repeat host hits), prefer table-driven tests and GoMock fakes.
- **SvelteKit / TypeScript** – Strict TypeScript, Svelte 5 Runes only (`$state` / `$derived` / `$effect`), TailwindCSS v4, Vitest + Testing Library; use `bun run test` (not bare `bun test`).
- **Python (FastAPI)** – Dependency injection via containers, async handlers, pytest + `pytest-asyncio`, maintain golden datasets for LLM prompt regressions, sanitize LLM outputs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaikei-e/Alt](https://github.com/Kaikei-e/Alt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
