---
trigger: always_on
description: Agent instructions for every coding agent (Claude Code, Codex, Cursor, Copilot, Gemini CLI). Each `CLAUDE.md` in the repo is a one-line `@AGENTS.md` stub so Claude Code loads the same file. Edit `AGENTS.md`, never the stub.
---

# AGENTS.md

Agent instructions for every coding agent (Claude Code, Codex, Cursor, Copilot, Gemini CLI). Each `CLAUDE.md` in the repo is a one-line `@AGENTS.md` stub so Claude Code loads the same file. Edit `AGENTS.md`, never the stub.

## Core technologies
- **Backend**: Go 1.24+ (see `server/AGENTS.md` for details)
- **Dashboard**: React + TanStack Router (see `web/dash0/AGENTS.md` for details)
- **Infrastructure**: Docker Compose with PostgreSQL for monitoring data storage
- **Monitoring**: Multi-protocol ping/health checking with distributed worker system
- **Docs site**: Docusaurus in `web/docs/` (baseUrl `/docs/`), embedded in the Go binary.
  - Served at `/docs` on every host (like `/d`, `/s`). `docs.solidping.io` redirects its root into `/docs` (`server.docs_host` / `SP_DOCS_HOST`).
  - The API reference is generated at build from `server/internal/app/openapi/openapi.yaml`. The OpenAPI explorer is at `/openapi`.
  - `/docs/changelog` is generated at build from the root `CHANGELOG.md` (entry conventions: `wiki/conventions/changelog.md`).
  - `llms.txt` / `llms-full.txt` come from `docusaurus-plugin-llms`. They are served at `/docs/` and at the root path `/` (same embedded file).
  - The marketing site (`www.solidping.io`) is the separate `solidping-website` repo. Internal engineering notes live in `wiki/`.
  - Never put competitor comparisons in the docs site. They go in `wiki/competitors/` (one `{name}.md` each, indexed in `wiki/README.md`).
  - The only competitor-facing pages in `web/docs/` are the `migrate-from-*.md` import guides.

## Development workflow
Laptop only: if a server already runs on port 4000, apply code changes directly (`make dev` / `make dev-test` hot-reload). `rtk` prefixes, the k8xp VPN and `gopass` are user-global setup, not part of the repo. Sandboxes and Claude cloud sessions: see [Without Docker](#without-docker-cloud-sessions-ci-like-sandboxes).

1. Start infrastructure: `docker-compose up -d`
2. Run everything: `make dev` (backend + dash0 + status0 with hot reload)
2b. Local env: `make dev` sources `dev.priv.env` (git-ignored, shell `KEY=value` lines) when it exists, e.g. the `SP_AI_*` variables. A git worktree without one uses the main checkout's. Read secrets there with `$(gopass show -o <path>)`.
3. Test mode: `make dev-test` (same but with `SP_RUNMODE=test`)
4. Database changes: add migrations, then `make migrate`

Dev logs live in `logs/*.log` (`backend.log`, `dash0.log`, `status0.log`), size-rotated with `.1`/`.2` suffixes (~20 MB cap); all three processes run as children of the `server/cmd/devloop` supervisor, so Ctrl-C stops everything.

### Without Docker (cloud sessions, CI-like sandboxes)
No Docker or Postgres is needed. SQLite is the default database. Full runbook: [wiki/runbooks/claude-cloud.md](wiki/runbooks/claude-cloud.md).

- Once per fresh sandbox: `scripts/cloud-setup.sh` (verifies Go, installs bun and golangci-lint into `~/.local/bin`, `make deps`, builds the frontend).
- Smoke-test server (run it in the background, then `curl localhost:4000/api/mgmt/version`): `cd server && SP_RUNMODE=test SP_DB_TYPE=sqlite SP_DB_DIR=$(mktemp -d) go run . serve`. Test-mode login is `test@test.com` / `test` / org `test`.
- Before pushing from a sandbox run `make lint` and `make test`. Leave `make test-postgres`, `make test-slow` and the full Playwright suite to CI.

### Key Makefile targets
| Target | Purpose |
|---|---|
| `make build` | Build everything |
| `make dev` | Hot-reload backend + dash0 + status0 |
| `make dev-test` | Same, with `SP_RUNMODE=test` |
| `make dev-saas` | Same, in SaaS mode — pairs with `../solidping-billing` `make dev` (see SaaS mode below) |
| `make test` | Run backend tests (`-short`: SQLite only, skips every Postgres suite) (`wiki/testing/test-layers.md`) |
| `make test-postgres` | Run the Postgres test layer (non-short, `-p 1`, `SP_TEST_REQUIRE_POSTGRES=1`) — exactly what the `backend-postgres` CI job runs, ~12 min |
| `make test-slow` | Run the `slowtests` build-tagged layer (live network + Docker); nightly in CI |
| `make test-dash` | Run dash0 Playwright tests |
| `make lint` | Lint backend + dash |
| `make fmt` | Format all code |
| `make migrate` | Run database migrations |
| `make bench-checks` | Benchmark check throughput (SQLite + Postgres) |
| `make bench-memory` | Measure memory precisely — fresh server per repetition, fixed sampling protocol, inter-run spread, `--compare` against a saved baseline. `BENCH_MEM_MODE=docker` (with `make bench-memory-image`) is the authoritative run; the default local mode is labelled non-authoritative in its own report. See `wiki/runbooks/memory-profiling.md` §5 |

## Default credentials

| Mode | Email | Password | Org |
|---|---|---|---|
| Normal | `admin@solidping.io` | `solidpass` | `default` |
| Test (`SP_RUNMODE=test`) | `test@test.com` | `test` | `test` |

Rules:
- The seeded `admin@solidping.io` has `users.must_change_password`. This applies to every fresh database, including `make dev`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fclairamb/solidping](https://github.com/fclairamb/solidping) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
