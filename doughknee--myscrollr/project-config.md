---
trigger: always_on
description: Operational guide for AI coding agents working in this repository.
---

# AGENTS.md

Operational guide for AI coding agents working in this repository.

## Project Overview

MyScrollr aggregates financial market data, sports scores, RSS feeds, and Yahoo Fantasy Sports. Tauri desktop app (primary product), React marketing website, Go gateway API, and independent channel services. Infrastructure: PostgreSQL, Redis, Logto (auth), Sequin (CDC), Stripe (billing). Deployed on DigitalOcean Kubernetes (DOKS) with images stored in DigitalOcean Container Registry (DOCR). See `k8s/` for manifests and `.github/workflows/deploy.yml` for the build-and-deploy pipeline.

## Repository Layout

Monorepo — each component is independently deployable with its own dependencies:

- `api/` — Core API (Go 1.25, Fiber v2). Internal packages under `api/internal/`; `api/core` wires them into one binary
- `myscrollr.com/` — Marketing website + auth/billing (React 19, Vite 7, TanStack Router, Tailwind v4)
- `desktop/` — Tauri v2 desktop app (React 19, Vite 7, TanStack Router + Query, Tailwind v4, Rust backend) — **primary product**
- The finance, sports, rss, and predictions Go APIs were folded into `api/internal/ingestread/` (ADR-0002); fantasy is the one remaining discovered channel service
- `channels/{finance,sports,rss,predictions}/service/` — Rust ingestion services (independent crates, edition 2024; predictions holds the Kalshi credentials and WS sweep)
- `channels/fantasy/api/` — Fantasy Go API (Yahoo OAuth2, Go-native sync, no Rust service)

## Running it locally

**Two commands from a fresh clone.** Everything below assumes you did this.

```sh
make setup   # generate every .env file (once)
make up      # start the whole backend in Docker, wait until healthy
```

`make` on its own prints every command, grouped. `make doctor` diagnoses a
broken machine and names the fix for each problem.

You need **Docker Desktop**, **Node 22+** and **make**. You do **not** need a
Go or Rust toolchain — the backend compiles inside its containers. Do not tell
a user to install Go or Rust to run this project.

| What | Where | Port |
|---|---|---|
| Postgres, Redis | Docker | 5432, 6379 |
| Core API | Docker | **18080** |
| Fantasy API | Docker | 8084 |
| finance / sports / rss ingesters | Docker | 3001 / 3002 / 3004 |
| predictions ingester (opt-in) | Docker | 3005 |
| Marketing site | native — `make web` | 3000 |
| Desktop app | native — `make desktop` | — |

**Core is published on 18080, not 8080.** Steam's CEF debugger claims
localhost:8080 on Windows. Inside the compose network services still reach core
on 8080; anything on the host uses 18080.

**Editing backend code needs no command.** Each container runs a file watcher
against bind-mounted source (`air` for Go, `cargo watch` for Rust), so saving a
`.go` or `.rs` file rebuilds that one service in place. `make rebuild` is only
for dependency changes (`go.mod`, `Cargo.toml`). `make logs svc=core-api` tails
one service; `make down` stops and keeps data; `make reset` wipes it.

Full runbook, including the Windows-specific traps: `docs/LOCAL_SETUP.md`.

## Build, Lint, Test Commands

### Website (`myscrollr.com/`)

```sh
npm run dev          # Vite dev server on port 3000
npm run build        # vite build && tsc (includes type-checking)
npm run check        # prettier --write . && eslint --fix (run before committing)
npm run lint         # eslint (no flags — pass your own, e.g. npm run lint -- --fix)
npm run format       # prettier (no flags — e.g. npm run format -- --write .)
```

### Desktop (`desktop/`)

```sh
npm run dev          # Vite frontend only on port 5174
npm run build        # vite build && tsc --noEmit (includes type-checking)
npm run tauri:dev    # Full Tauri dev (Vite + Rust backend)
npm run tauri:build  # Production build (native binary)
```

### Backend (Go and Rust)

**You don't run these by hand.** `make up` builds and runs every backend
service in Docker, and the containers hot-reload on save. There is no
supported native workflow and no host toolchain requirement.

To compile-check or test a specific service without a local toolchain, use its
container: `make shell svc=core-api` then `go build ./...`, or
`make shell svc=rss-service` then `cargo check`.

### Tests

- **TypeScript** (Vitest): All: `npx vitest run`. File: `npx vitest run path/to/file.test.ts`. Single: `npx vitest run -t "test name"`.
- **Go**: All: `go test ./...`. File: `go test ./path/to/pkg`. Single: `go test -run TestName ./path/to/pkg`.
- **Rust**: All: `cargo test`. Single: `cargo test test_name`.

Go integration tests (GDPR purge cascade, Stripe webhook idempotency, fantasy's schema contract) need a real Postgres and gate on `TEST_DATABASE_URL` — they skip when it's unset, so plain `go test ./...` always works without a database. To run them locally, point the variable at a scratch database (the tests apply the repo's migrations and truncate the tables they touch — never use a database with real data):

```sh
TEST_DATABASE_URL="postgres://postgres@127.0.0.1:5432/scrollr_test?sslmode=disable" go test ./...
```

### CI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doughknee/myscrollr](https://github.com/doughknee/myscrollr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
