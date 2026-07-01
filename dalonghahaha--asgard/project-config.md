---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Asgard — a single-binary distributed job management platform that unifies **resident processes (app)**, **cron-like scheduled jobs (job)**, and **one-shot timed jobs (timing)**. One Go binary dispatches via cobra to subcommands `web` / `master` / `agent` / `guard` / `cron` / `debug`. Three node roles cooperate over gRPC: **web** is the control plane, **master** is the registry/state sink, **agent** is the worker that actually spawns OS processes.

## Current state — read this first

The project is **mid-migration** (TASKS Phase 0~5, see `doc/TASKS.md`):

- **Phase 0** (planning) is complete — Vue 3 + Vite + TS + Element Plus + Pinia + axios + SSE chosen; `/api/v1` is the API prefix; auth is dual-track `Authorization: Bearer <jwt>` + legacy DES cookie.
- **Phase 1** (backend API) is in progress — `/api/v1` group is registered (`web/router.go` + `web/routers/api_router.go`); 11 resource groups + SSE handlers are wired; old HTML controllers have been moved to `web/legacy/` and `doc/legacy-templates/`, which carry `//go:build ignore` and are **not for new code**.
- **Phase 2-5** (frontend migration → cutover) are still in flight.

**Rule:** Before any non-trivial change, scan `doc/TASKS.md` §3 to avoid stepping on a sibling task. Start → mark `[/]`, finish/blocked → mark `[x]/[-]` and append a line to §4.

## Build, run, test

```sh
# Build (single binary, CGO off; uses Go 1.16 toolchain — see "known traps" below)
go build -o Asgard .

# Run a node (each needs a conf/app.yaml — see README §部署 for templates)
./Asgard web    -c conf    # JSON API on :12345 (no HTML since cutover)
./Asgard master -c conf    # gRPC on :9527
./Asgard agent  -c conf    # gRPC on :27149
./Asgard guard  -c conf [-s runtime/asgard_guard]   # supervisor-style, Unix socket
./Asgard cron   -c conf [-s runtime/asgard_cron]   # crontab-style, Unix socket
./Asgard debug ...                                    # mail / RPC tools
./Asgard agent status [-p 27149]                      # inspect local agent

# Tests (Go) — there is only one in-tree test file today:
go test ./web/utils/ -run TestIssueAndParseToken -v
go test ./web/utils/ -run TestParseToken -v
# (no project-wide test runner or coverage gate exists; add new tests per-package)

# Frontend (web-admin/)
cd web-admin
npm install
npm run build          # outputs web-admin/dist/, packaged by .goreleaser.yml
npm run test           # Vitest unit
npm run e2e            # Playwright (config: playwright.config.ts)
npm run dev            # Vite dev server; /api/* proxied to VITE_BACKEND_TARGET (default :12345)

# Regenerate gRPC stubs (requires legacy protoc-gen-go v1.x; new toolchains need adjustment)
bash scripts/protoc.sh

# Snapshot release (multi-platform: darwin/linux × i386/amd64/arm/arm64)
goreleaser release --snapshot --clean
```

There is **no** `Makefile`, `golangci-lint` config, or Go test harness in the repo — don't invent one before checking with the user.

## High-level architecture

```
                ┌────────────┐   gRPC    ┌──────────┐   gRPC   ┌────────┐
   Browser  ──▶ │  Asgard    │ ────────▶ │  master  │ ───────▶ │ agent  │──▶ os/exec
  (web-admin/   │  web       │           │ (registry│          │(managers
   SPA + SSE)   │  (Gin API) │ ◀───────  │ + state) │ ◀──────  │ + runtimes)
                └─────┬──────┘  gRPC/JOB └────┬─────┘  archive/ └────────┘
                      │                      │        monitor/
                      ▼                      ▼        exception
                ┌────────────┐         ┌──────────────────┐
                │  MySQL     │         │  Redis (cache)   │
                │ (incl.     │         │  (users/groups)  │
                │  monitors_ │         └──────────────────┘
                │  YYYYMM)   │
                └────────────┘
```

- **`web/`** — Gin server, JSON-only after cutover. `web/router.go` mounts `routers.SetupAPIRouter` under `/api/v1`; the rest of the tree lives in `web/middlewares/` (CORS + `APIAuth` + `APIAuthAdmin`) and `web/controllers/api_*.go` (one file per resource). Response helpers in `web/utils/respose.go`; SSE handlers in `api_sse.go`.
- **`cmds/<sub>/`** — cobra entry per subcommand. Each follows the same recipe: viper-load `conf/app.yaml` → register avenger components (`db/cache/logger/mail`) → init the relevant manager/controllers → `runtimes.Wait(...)` for SIGTERM/SIGINT.
- **`models/` + `services/` + `providers/`** — GORM persistence, business logic, and a service-singleton container (wires the 11 services in `providers/service.go` `init()`). All `WhereAndOrder / PageListbyWhereString` queries are **string-concatenated** (see "known traps").
- **`server/` + `clients/`** — gRPC server implementations and client wrappers. `clients/base.go` provides a Unix-socket dialer for `guard`/`cron`.
- **`managers/` + `runtimes/`** — agent-side lifecycle. `runtimes/cmd.go` defines the `Command` abstraction embedded by `App/Job/Timing`; `runtimes/monitor.go` holds the CPU/mem sampler; archive/exception types live alongside.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dalonghahaha/Asgard](https://github.com/dalonghahaha/Asgard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
