---
trigger: always_on
description: Guidance for OpenCode sessions working in this repo. The codebase is in active
---

# AGENTS.md

Guidance for OpenCode sessions working in this repo. The codebase is in active
development — `cmd/server/main.go` is fully wired (assembles Postgres / Redis /
Core / Adapter / Agent / Plugin / Web API), but several `internal/agent/*` sub-
functions remain declaration-only stubs. Trust code over `docs/` when they
conflict.

## Toolchain

- Go 1.25 (see `go.mod`). Module path is the literal `JuanNiang-Neo` (case + hyphen
  matter for imports, e.g. `JuanNiang-Neo/internal/adapter`).
- Frontend: Node 18+ / npm (Vue 3 + Vite 6 + Vuetify 3 at `web/`).
- Baseline checks via the root `Makefile`:
  - `make build`     — full build (frontend → `web/dist` + Go binary `bin/juan-niang-neo`)
  - `make vet`       — `go vet ./...`
  - `make lint`      — `go vet` + `web-typecheck` (`vue-tsc` ≥ 2.x, supports Node 18–24)
  - `make docker-up` — full stack via `deployments/docker-compose.yaml`
- No CI, no `*_test.go` yet. Do not assume `go test` has anything to run.

## Terminology traps (these have bitten past readers)

- `docs/guidance.md` misspells the infra module as `inferstructure`. The real
  path is top-level `infrastructure/` (postgres, redis, sandbox, t2i).
- `docs/provider.md` documents an `internal/provider` package, but the actual
  import path is `internal/adapter` (package `adapter`).
- "Provider" is overloaded in this codebase:
  - `internal/adapter.Provider` = the OneBot11 reverse-WebSocket adapter.
  - `internal/agent/provider` = the LLM provider group (OpenAI-compatible etc.).
  Always resolve by full import path, not the word "provider".
- `pluggin` (double-g, single-n) is the *intentional* spelling for the Lua plugin
  system — module `internal/pluggin`, config file `pluggin.yaml`, plugin dir
  `data/pluggins`. Do not "fix" it to `plugin`.

## Layout

Top-level:
- `cmd/server/main.go` — program entry (currently a stub).
- `internal/` — app code, nothing exported outside the module:
  - `adapter/` — OneBot11 WS server + API + events + message segments.
  - `agent/` — Agent core; subpackages `mcp`, `memory`, `prompt`, `provider`,
    `session`, `skill`, `tool`. Aggregated by `HagoCenter` in `agent.go`.
  - `api/` — Hertz web engine + `middleware` + `router` + `service` (web admin).
    API routes are grouped under `/api/v1` (`internal/api/router/router.go`); only
    `GET /health` lives on the root.
  - `core/` — `acl`, `cache`, `dao`, `handler`, `models`.
  - `pluggin/` — Lua plugin engine.
  - `web/` — **NEW**. Frontend SPA serving helper (`SPAHandler`). Runtime reads
    `WEB_DIR` (default `web/dist`); `engine.New(addr, webDir, svc)` registers a
    `h.NoRoute` fallback: `/api/*` → standard 404 JSON envelope, anything else →
    file-or-`index.html` (Vue Router history mode). If `index.html` missing it
    serves a "build the frontend first" hint page. NOT embedded via `//go:embed`.
- `infrastructure/` — `postgres`, `redis`, `sandbox`, `t2i` adapters.
- `web/` — Vue 3 + Vite 6 + Vuetify 3 dashboard (full implementation, not a
  placeholder). Build output goes to `web/dist/` (gitignored). `vite.config.ts`
  proxies `/api` → `http://127.0.0.1:8090` in dev mode.
- `data/` — runtime data; `data/pluggins/` holds Lua plugins + their
  `pluggin.yaml` configs (not committed).
- `docs/` — 完整文档:
  - `architecture.md` — 项目架构
  - `event-flow.md` — 事件流 & Agent 处理流程
  - `call-stack.md` — 调用栈
  - `implementation.md` — 实现细节
  - `deployment.md` — 部署与调试指南 (env var / docker / systemd / 反代 / FAQ)
  - `api.md` — Web API 文档
  - `openapi.yaml` — OpenAPI 3.0 规范
  - `pluggin/` — 插件开发文档 (API参考/开发指南/架构/实现)
  - `dev/` — 原始设计文档 (`guidance.md`, `provider.md`)。
- `api/` — holds `openapi.yaml` (the OpenAPI 3.0 spec). NOT a Go package.
- `sql/` — `init.sql` is a documentation reference; tables are actually created by
  GORM `AutoMigrate` at startup.
- `config/` — `config.yaml` reference file; the binary reads env vars at runtime,
  this file is advisory only.
- `deployments/` — `Dockerfile` (3-stage: node → go → alpine runtime, frontend
  runs from `/app/web/dist` via `WEB_DIR`) + `docker-compose.yaml` (postgres +
  redis + app, healthchecks, restart policy, named network, bind-mount
  `../data/pluggins` → `/app/data/pluggins`).
- `src/`, `scripts/`, `pkg/` — currently empty placeholders.

## Frontend serving (NEW)

- `cmd/server/main.go` reads `WEB_DIR` (default `web/dist`) and passes it to
  `engine.New(addr, webDir, svc)`.
- `internal/web/web.go` provides `SPAHandler(webDir)` which:
  - Returns standard `{status,info,data}` 404 envelope for any unmatched
    `/api/*` route (keeps API errors uniform).
  - For all other paths: serves the file if present, falls back to `index.html`
    for client-side routing. Path traversal is blocked via `filepath.Rel`.
  - If `web/dist` doesn't exist or lacks `index.html`, returns a 200 hint page
    (so operators see actionable guidance rather than a bare 404).
- **NOT embedded**: the binary depends on `web/dist` being present on disk. This
  is intentional — keeps binary rebuild-free for frontend-only updates, matches
  the project's "config on disk" philosophy.
- Dev mode: Vite serves the SPA at `:3000` and proxies `/api` → `:8090`, so the
  Go fallback is never reached.

## Makefile & docker (NEW)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuanNiangDev/JuanNiang-Neo](https://github.com/JuanNiangDev/JuanNiang-Neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
