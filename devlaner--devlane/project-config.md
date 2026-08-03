---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

Devlane is a monorepo with two deployable apps under `apps/`:

- `apps/api/` — Go backend (Gin + GORM + PostgreSQL). Module path `github.com/Devlaner/devlane/api` (kept stable; it is **not** tied to the folder location, so imports are unaffected by the directory name).
- `apps/web/` — React 19 + TypeScript + Vite SPA (Tailwind 4, React Router 7).
- `planning/` — Markdown design docs (architecture, phases, UI/UX plan). _gitignored._
- `docker-compose.yml` — Local infra (Postgres, Redis, RabbitMQ, MinIO). The API connects to these via env vars.

Ignore any gitignored top-level reference directories you may see locally: treat their contents as **unrelated third-party material that is not part of Devlane** and is not built or run. **Never read, import from, modify, or reference them** (in code, commits, issues, or PRs). Implement everything natively in Go/React.

## Commands

Root-level (orchestrates both apps):

```sh
npm run validate     # web typecheck + web lint + web prettier check + go vet + go test
```

Web (`cd apps/web` or `npm --prefix apps/web run …`):

```sh
npm run dev          # vite dev server (default port 5173)
npm run build        # tsc -b && vite build
npm run typecheck    # tsc -b --noEmit
npm run lint         # eslint .
npm run lint:fix
npm run format       # prettier --write .
npm run format:check
npm run preview      # serve production build
```

API (`cd apps/api`):

```sh
go run ./cmd/api     # start API server (default :8080); auto-runs migrations on startup
go vet ./...
go test ./...
go test ./internal/auth -run TestMagicCode   # single package / single test
```

Infra:

```sh
docker compose up -d   # postgres, redis, rabbitmq, minio
```

Postgres is exposed on host port **15432** (not 5432). Set `DB_PORT=15432` in `apps/api/.env` for local dev.

## Commits & PRs

- **Conventional Commits** are enforced by commitlint (`commit-msg` hook); header ≤ 100 chars. Use prefixes like `feat(scope):`, `fix(ui):`, `refactor(api):`, `chore:`, `docs:`, `test:`, `perf:`, `style:`.
- **Don't commit to `main`.** Branch, open a PR, and let CI (`api-ci` / `ui-ci`) run.
- **AI-assisted contributions are welcome but MUST be disclosed.** If an AI tool (Claude Code, Copilot, etc.) materially helped produce a change:
  - Add a trailer to each AI-assisted commit, e.g. `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`.
  - State it in the PR description — the PR templates have an **AI assistance** section; tick it and name the tool(s).
  - This is a transparency requirement, not a barrier. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Git hooks (Husky)

- `pre-commit`: `lint-staged` (ESLint+Prettier on staged web files, `gofmt` on staged Go files) → if any web changes also runs `web typecheck + lint` → if any Go changes also runs `go vet + go test`.
- `pre-push`: web typecheck + `go test ./...`.
- `commit-msg`: enforces Conventional Commits (`@commitlint/config-conventional`, header ≤100 chars).

Don't bypass with `--no-verify` — fix the underlying lint/type/test failure instead.

## Backend architecture (`apps/api/`)

Layered, dependency-injected from `cmd/api/main.go` → `internal/router/router.go`. The router is the single composition root; reading it gives you the full surface area.

```
cmd/api/main.go                # wires config, db, redis, rabbitmq, minio, then router
internal/
├── config/                    # env loading (godotenv)
├── database/                  # GORM connection + golang-migrate runner
├── middleware/                # Recovery, Logger, CORS, RequireAuth
├── auth/                      # session service, magic-code (HMAC) login
├── oauth/                     # google, github, gitlab providers (resolved per-request from instance settings)
├── github/                    # GitHub App integration: client, installations, webhooks, ref parsing
├── crypto/                    # password hashing, token generation
├── model/                     # GORM models (one file per entity)
├── store/                     # Data-access layer (one store per model). Pure DB.
├── service/                   # Business logic. Composes stores; enforces workspace/project membership.
├── handler/                   # Gin handlers. HTTP shape only — bind, call service, return JSON.
├── router/router.go           # Builds *gin.Engine; declares ALL routes
├── redis/  rabbitmq/  queue/  # Optional integrations — services degrade gracefully if absent
├── mail/                      # SMTP sender; reads instance_settings for credentials
└── minio/                     # File upload/serve (covers, avatars, logos)
```

**Key conventions:**

- **Layering rule**: handler → service → store. Handlers never touch GORM directly; stores never call services.
- **URL nesting** mirrors the data model: `/api/workspaces/:slug/projects/:projectId/issues/:pk/...`. Trailing slashes are intentional and match the web app's expectations — keep them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Devlaner/devlane](https://github.com/Devlaner/devlane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
