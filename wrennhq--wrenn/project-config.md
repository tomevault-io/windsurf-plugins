---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Wrenn is an open-source, self-hosted dev environment platform. Users spin up isolated sandboxes (Cloud Hypervisor microVMs), run code inside them, and get output back via SDKs. Fast boot, persistent state, and a single agent binary on each host you own.

## Build & Development Commands

All commands go through the Makefile. Never use raw `go build` or `go run`.

```bash
make build              # Build all binaries → builds/
make build-cp           # Control plane only
make build-agent        # Host agent only
make build-envd         # envd static binary (Rust, musl, verified statically linked)
make build-frontend     # SvelteKit dashboard → frontend/build/ (served by Caddy)

make dev                # Full local dev: infra + migrate + control plane
make dev-infra          # Start PostgreSQL + Prometheus + Grafana (Docker)
make dev-down           # Stop dev infra
make dev-cp             # Control plane with hot reload (if air installed)
make dev-frontend       # Vite dev server with HMR (port 5173)
make dev-agent          # Host agent (sudo required)
make dev-envd           # envd in debug mode (port 49983)

make check              # fmt + vet + lint + test (CI order)
make test               # Unit tests: go test -race -v ./internal/...
make test-integration   # Integration tests (require host agent + Cloud Hypervisor)
make fmt                # gofmt and rust fmt
make vet                # go vet
make lint               # golangci-lint

make migrate-up         # Apply pending migrations
make migrate-down       # Rollback last migration
make migrate-create name=xxx  # Scaffold new goose migration (never create manually)
make migrate-reset      # Drop + re-apply all

make generate           # Proto (buf) + sqlc codegen
make proto              # buf generate for proto dirs
make tidy               # go mod tidy
```

Run a single test: `go test -race -v -run TestName ./internal/path/...`

## Architecture

```
User SDK → HTTPS/WS → Control Plane → Connect RPC → Host Agent → HTTP/Connect RPC over TAP → envd (inside VM)
```

**Three binaries:**

| Binary | Language | Entry point | Runs as |
|--------|----------|-------------|---------|
| wrenn-cp | Go (`git.omukk.dev/wrenn/wrenn`) | `cmd/control-plane/main.go` | Unprivileged |
| wrenn-agent | Go (`git.omukk.dev/wrenn/wrenn`) | `cmd/host-agent/main.go` | `wrenn` user with capabilities (SYS_ADMIN, NET_ADMIN, NET_RAW, SYS_PTRACE, KILL, DAC_OVERRIDE, MKNOD) via setcap; also accepts root |
| envd | Rust (`envd-rs/`) | `envd-rs/src/main.rs` | PID 1 inside guest VM |

envd is a standalone Rust binary (Tokio + Axum + connectrpc-rs). It is completely independent from the Go module — the only connection is the protobuf contract. It compiles to a statically linked musl binary baked into rootfs images.

**Key architectural invariant:** The host agent is **stateful** (in-memory `boxes` map is the source of truth for running VMs). The control plane is **stateless** (all persistent state in PostgreSQL). The reconciler (`internal/api/reconciler.go`) bridges the gap — it periodically compares DB records against the host agent's live state and marks orphaned sandboxes as "stopped".

### Control Plane

**Internal packages:** `internal/api/`, `internal/email/`

**Public packages (importable by cloud repo):** `pkg/config/`, `pkg/db/`, `pkg/auth/`, `pkg/auth/oauth/`, `pkg/auth/session/`, `pkg/auth/session/middleware/`, `pkg/scheduler/`, `pkg/lifecycle/`, `pkg/channels/`, `pkg/audit/`, `pkg/service/`, `pkg/events/`, `pkg/id/`, `pkg/validate/`

**Extension framework:** `pkg/cpextension/` (shared `Extension` interface + `ServerContext` + hook interfaces), `pkg/cpserver/` (exported `Run()` entrypoint with functional options for cloud `main.go`)

The cloud repo imports this module as a Go dependency and calls `cpserver.Run(cpserver.WithExtensions(myExt))`. An extension always implements:
- `RegisterRoutes(r chi.Router, sctx ServerContext)` — adds HTTP routes.
- `BackgroundWorkers(sctx ServerContext) []func(context.Context)` — starts long-running goroutines.

It can optionally implement any of these hook interfaces (the OSS server type-asserts at startup):
- `MiddlewareProvider` — `Middlewares(sctx) []func(http.Handler) http.Handler`, applied before OSS routes so cloud middleware can wrap them (e.g. billing gates).
- `AuthHook` — `OnSignup` (synchronous, error aborts the request with 500 `signup_hook_failed`), `OnLogin`, `OnAccountSoftDelete`, `OnAccountHardDelete` (the last three log + ignore errors). OnSignup fires after team provisioning in both email-activate and OAuth-new-signup paths.
- `SandboxEventHook` — `OnSandboxEvent(ctx, SandboxEvent)`, invoked from the unified Redis stream consumer for capsule create/pause/resume/destroy success events. Hook errors leave the message un-acked so it will be redelivered; hooks must be idempotent.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wrennhq/wrenn](https://github.com/wrennhq/wrenn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
