---
trigger: always_on
description: Ollanta is a multi-language static analysis platform in Go. It scans source code, reports quality issues (bugs, code smells, vulnerabilities), computes metrics, and evaluates configurable quality gates.
---

# Ollanta â€” Project Guardrails

Ollanta is a multi-language static analysis platform in Go. It scans source code, reports quality issues (bugs, code smells, vulnerabilities), computes metrics, and evaluates configurable quality gates.

**Two main components:**
- **Scanner** (`ollantascanner`) â€” CLI that discovers files, parses, applies rules, produces JSON/SARIF reports. Optional local web UI on port 7777.
- **Server** (`ollantaweb`) â€” receives scan reports, tracks issues across scans, stores history, evaluates quality gates, REST API on port 8080.

**Supported languages:** Go (native `go/ast`), JavaScript, TypeScript, Python, Rust (tree-sitter).

## Architecture

This is a **Go workspace** (`go.work`) with 9 modules. The root `Makefile` covers 8 of 9 modules (all except `adapter/`, which requires a running Postgres instance).

Hexagonal architecture (ports & adapters) with three rings:

| Ring | Modules | Deps allowed |
|------|---------|--------------|
| **Inner** | `domain/` | Go stdlib only |
| **Middle** | `application/` | `domain/` + `ollantacore/` only |
| **Outer** | `adapter/`, `ollantaweb/`, `ollantastore/` | Everything |

Supporting modules (`ollantacore/`, `ollantaparser/`, `ollantarules/`, `ollantascanner/`) provide shared types, parsing, rules, and the CLI scanner.

### CGo Boundary

Only `ollantaparser` has CGo (tree-sitter C library). The domain layer uses `any` for tree-sitter types to stay CGo-free. `ollantaweb` must NEVER import `ollantaparser` or `ollantarules` transitively â€” its Dockerfile builds with `CGO_ENABLED=0`.

### Adapter Bridge Pattern

`adapter/secondary/rules/bridge.go` converts between legacy types (`ollantacore/domain.Issue`) and hexagonal types (`domain/model.Issue`). Always use the bridge â€” never mix types directly.

## Module Layout

| Module | Purpose | CGo |
|--------|---------|-----|
| `domain/` | Pure models, port interfaces, domain services | No |
| `application/` | Use cases: scan, ingest, analysis | No |
| `adapter/` | HTTP, OAuth, Postgres, Parser, Rules bridge, Telemetry, Webhook | Yes* |
| `ollantacore/` | Shared types with type aliases to `domain/model` | No |
| `ollantaparser/` | Tree-sitter C bindings â€” **only true CGo module** | **Yes** |
| `ollantarules/` | Rule registry, Go/tree-sitter sensors, metadata | Yes* |
| `ollantascanner/` | CLI entry point, file discovery, parallel executor | Yes* |
| `ollantastore/` | PostgreSQL repos (pgx/v5), search (ZincSearch/Postgres FTS) | No |
| `ollantaweb/` | REST server, ingestion, auth, webhooks (chi/v5) | No |

_*Transitive CGo via `ollantaparser`._

## Developer Setup

### Prerequisites
- Go 1.21+, CGo toolchain (gcc/clang or MSYS2 MinGW on Windows)
- Docker & Docker Compose
- Node.js (for scanner frontend build)

**Windows CGo note:** MSYS2 MinGW's `gcc` must be in `%PATH%`. Add `C:\msys64\mingw64\bin` to your user or system PATH. Without it, `CGO_ENABLED=0` and all `go-tree-sitter` types become unresolvable, breaking flycheck/gopls diagnostics in packages that import it.

### Commands

| Command | Description |
|---------|-------------|
| `make build` | Build all 8 non-adapter modules (CGo) |
| `make test` | Test all 8 non-adapter modules (CGo) |
| `make lint` | Lint 7 modules per-module (domain..ollantastore; excludes ollantaweb+adapter) |
| `make fmt` | Format source code |
| `make run` | Run scanner + local UI (overridable: `PROJECT_DIR`, `PROJECT_KEY`, `PORT`) |
| `make push` | Scan + push results to server |
| `make up` / `make down` | Start / stop the Docker server stack |
| `make clean` | Clean build artifacts |
| `make recreate` | Full destroy + rebuild server stack (volumes, images, cache) |
| `make smoke-local` | Local end-to-end smoke test (scanner → server) on port 18080 |

**Docker Compose profiles:**
- `docker compose --profile scanner up local-ui` â€” scanner local UI on 7777
- `docker compose --profile server up` â€” postgres + zincsearch + ollantaweb (8080)
- `docker compose --profile push run --rm push` â€” scan + push results to server

**Scanner push authentication:**

The push service sends results to `ollantaweb` via `POST /api/v1/scans`, which requires auth. A pre-shared scanner token avoids needing a user account:

| Variable | Where | Default (dev) |
|----------|-------|---------------|
| `OLLANTA_SCANNER_TOKEN` | `ollantaweb` (server side) | `ollanta-dev-scanner-token` |
| `OLLANTA_TOKEN` | `push` service (scanner side) | `ollanta-dev-scanner-token` |

For production, set both to the same strong random secret in `.env`. If `OLLANTA_SCANNER_TOKEN` is empty, scanner push falls back to regular JWT/API token auth.

### After Code Changes

**Scanner/rules (CGo modules):** `make build` â†’ `make test`

**Web / hexagonal modules (no CGo):**
```sh
go build ./domain/... ./application/... ./ollantastore/... ./ollantaweb/...
go test  ./domain/... ./application/... ./ollantastore/... ./ollantaweb/...
```

**Adapter module (CGo, needs Postgres for tests):**
```sh
go test ./adapter/...
```

**Server (`ollantaweb`):** changes take effect on rebuild: `docker compose --profile server build ollantaweb`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lobovit/Ollanta](https://github.com/lobovit/Ollanta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
