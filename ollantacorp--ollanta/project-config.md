---
trigger: always_on
description: Multi-language static analysis platform in Go. Two deliverables:
---

# Ollanta - Agent Guide

Multi-language static analysis platform in Go. Two deliverables:

- **Scanner** (`ollantascanner`) - CLI: discovers files, parses (Go via `go/ast`; JS/TS/Python/Rust via tree-sitter), applies rules, writes JSON/SARIF reports to `<project>/.ollanta/`. Optional embedded UI on port 7777.
- **Server** (`ollantaweb`) - REST API on port 8080: ingests scan reports (asynchronously, via background workers), tracks issues across scans, evaluates quality gates.

Deeper docs: `docs/architecture.md`, `docs/contributing.md`, `CONTRIBUTIONS.md` (fast operational checklist). Change proposals use OpenSpec under `openspec/changes/`.

## Modules

`go.work` lists 9 modules. The root Makefile builds/tests 8 of them (all but `adapter/`, which needs a live Postgres).

| Module | Purpose | CGo |
|--------|---------|-----|
| `domain/` | Models, ports, domain services. stdlib only. | No |
| `application/` | Use cases (scan, ingest). Internal imports: `domain/` + `ollantacore/` only. | No |
| `adapter/` | Hexagonal adapters (`secondary/`: parser, rules bridge, telemetry). | Yes* |
| `ollantacore/` | Shared legacy types + `rulecatalog` (CGo-free rule catalog used by `ollantaweb`). | No |
| `ollantaparser/` | Tree-sitter bindings. **Only true CGo module.** | **Yes** |
| `ollantarules/` | Rule registry, sensors, embedded JSON metadata. | Yes* |
| `ollantascanner/` | CLI entrypoint (`cmd/ollanta`), discovery, executor, local UI server. | Yes* |
| `ollantastore/` | Postgres repos (pgx/v5), search (ZincSearch). | No |
| `ollantaweb/` | REST server (chi/v5), auth, ingest, webhooks. | No |

_*Transitive CGo via `ollantaparser`._

**Satellite modules outside `go.work`** - easy to miss; NOT covered by `make` or CI:

- `ollantaengine/` - quality gates, new-code tracking, summarizer. `ollantaweb` depends on it via `replace`. Run its tests yourself from inside the dir (set `$env:GOWORK='off'` if go complains it is not in the workspace).
- `tests/e2e/` - full scanner -> server -> ingest pipeline test with its own Makefile (`make test`). Builds the scanner, boots a disposable compose stack, skips when Docker is unavailable.

The server image (`ollantaweb/Dockerfile`, built with `CGO_ENABLED=0`) produces 5 binaries: `ollantaweb` (API), `ollantaworker` (scan-job processing - without it, pushed scans never complete), `ollantaindexer` (search projection), `ollantawebhookworker` (webhook delivery), `ollantamigrate` (one-shot schema migration).

## Commands

| Command | Does |
|---------|------|
| `make build` / `make test` | 8 workspace modules (excludes `adapter/`), CGO on |
| `make lint` | golangci-lint per-module on 7 modules - excludes `ollantaweb/` (entire dir excluded in `.golangci.yml`) and `adapter/` |
| `make run` / `run-bg` / `stop` | Scan + local UI on :7777 (vars: `PROJECT_DIR`, `PROJECT_KEY`, `PORT`) |
| `make push` | Scan + push with `-server-wait`. **Exits 3 when the server-side quality gate fails** - that is a gate failure, not an ingestion error |
| `make up` / `down` / `recreate` / `logs` | Compose `server` profile: postgres + zincsearch + web + 3 workers |
| `make smoke-local` | PowerShell end-to-end smoke (scanner -> server) on port 18080 (`SMOKE_BACKEND_PORT` to override). Preserves temp project + server log on failure |
| `make swagger` | Regenerates `ollantaweb/docs/` from swag annotations |
| `make release` / `release-dry-run` | Cross-compile the scanner for 5 platforms |

Traps:

- **Never run `golangci-lint` at the workspace root** - each module has its own `go.mod`. Run per-module, like the Makefile does.
- Windows: the Makefile prepends `C:\msys64\mingw64\bin` to PATH **for its own targets only**. For direct `go` / `golangci-lint` on CGO packages: `$env:CGO_ENABLED='1'; $env:PATH = 'C:\msys64\mingw64\bin;' + $env:PATH`.
- Scanner UI (TypeScript): `cd ollantascanner/server/static && npm test && npm run build`.
- Server SPA (vanilla JS): `node --test app.test.js` in `ollantaweb/api/static/`.
- Web e2e (`ollantaweb/e2e/`, Playwright) needs the full `server` profile up **including `ollantaworker`** - ingestion is async, so issue tests hang or show empty states without it. Seeded login: `admin` / `admin`.

## After a change, validate

| Changed area | Run |
|--------------|-----|
| Scanner / rules / parser (CGO) | `make build && make test` |
| Web / hexagonal (no CGO) | `go build ./domain/... ./application/... ./ollantastore/... ./ollantaweb/...` and `go test` on the same |
| `ollantaengine/` | `go test ./...` inside that dir (not covered by make/CI) |
| `adapter/` | `go test ./adapter/...` with `DATABASE_URL` pointing at Postgres |
| Server behavior | `docker compose --profile server build ollantaweb`, then `make smoke-local` |
| Scanner UI `src/` | `npm run build` and commit `dist/` (embedded in the binary) |

`ollantastore/postgres` tests **skip silently** unless `DATABASE_URL` (or `OLLANTA_TEST_DATABASE_URL`) is set - a green `make test` does not mean the DB layer was exercised.

## CI

5 jobs on push/PR to `main`: `test-scanner` (CGO, `-race`), `test-web` (`CGO_ENABLED=0` + Postgres service), `test-adapter` (CGO + Postgres), `lint` (golangci-lint v2 on the 4 scanner modules only, plus gofmt check), `docker-build` (scanner + server images).

## Boundaries (enforced in review)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ollantacorp/Ollanta](https://github.com/ollantacorp/Ollanta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
