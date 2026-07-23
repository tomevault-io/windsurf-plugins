---
trigger: always_on
description: LLM Ops platform for evaluation, observability, and optimization of AI agents and pipelines.
---

# LangWatch

LLM Ops platform for evaluation, observability, and optimization of AI agents and pipelines.

## Before You Implement Anything

**Check `specs/` first.** Feature files ARE the requirements.

```
ls specs/                    # Find relevant subdirectory
cat specs/foo/bar.feature    # Read the scenarios
```

If no feature file exists for your task, create one before writing code.

**For frontend work, read the UX docs first.** Before any non-trivial frontend change (anything beyond a specific, targeted tweak the user spelled out), read the relevant pattern docs under `dev/docs/best_practices/` so you extend existing patterns instead of reinventing them. The UI ones: `react.md`, `drawers.md`, `row-actions-overflow-menu.md`, `selection-action-bar.md`, `scope-selector-and-badges.md`. If the surface you are building has no doc yet, write one as part of the change.

## Development Environment

`make quickstart` is the single entry point. It asks what you're working on and starts only the services you need, overriding only the URLs whose services are local. Your `langwatch/.env` is the source of truth for everything else.

### Local dev by hostname — thuishaven / portless (recommended)

Stop juggling ports. Opt in with `pnpm dev:haven` and traffic routes through
**`haven`** (the Go orchestrator in `tools/thuishaven`, binary `cmd/haven`), which
gives every worktree's services a
stable hostname via the [portless](https://github.com/vercel-labs/portless) proxy —
`app|gateway|nlp.<slug>.langwatch.localhost`, where `<slug>` is the worktree's own
directory name, sanitised (a checkout at `.../worktrees/portless` is the `portless`
stack). The app and its API share one origin — open `app.<slug>...` for the UI,
hit `app.<slug>.../api` for the API. `.localhost` resolves to loopback natively,
so there is no `/etc/hosts`, DNS, or sudo for name resolution, and two worktrees
can never collide.

Hostname routing is **opt-in** — `pnpm dev` uses the plain `PORT`+offset scheme;
`pnpm dev:haven` (or `make haven up`) routes through haven.

```bash
make haven setup        # one-time: install/verify portless (443, trusted CA)
make haven install      # optional: go install so plain `haven ...` works everywhere
pnpm dev:haven          # == make haven up (registers hostnames, supervises the stack)
make haven list         # which worktree runs what (all stacks)
make haven doctor       # proxy / daemon / observability health
```

Open `https://langwatch.localhost` for the cross-worktree dashboard;
`observability.langwatch.localhost` proxies the local Grafana LGTM stack;
`telemetry.langwatch.localhost` fans OTLP out to every running stack. haven's
resolved config lands in `langwatch/.env.portless` (loaded last with
`override: true` so it beats `.env`). Agent-driving haven? Add `--agent` (or
`HAVEN_AGENT=1`) for plain, token-free output; `haven list --json` is
machine-readable. See `tools/thuishaven/README.md`.

```bash
make quickstart                        # Interactive preset picker
make quickstart all-local              # Local CH + PG + Redis + app + workers, no NLP (fast iteration default)
make quickstart all-local-nlp          # all-local + nlpgo + langevals
make quickstart dev-storage            # Local DBs + workers, stored-objects -> dev S3 (runtime-storage-dev)
make quickstart dev-infra              # Local app + redis + workers compose; shared dev for PG/CH/NLP/S3
make quickstart frontend-only          # No compose, fastest — UI / design work
make quickstart migration              # postgres + clickhouse on host ports for prisma migrate (no app, no workers)
make quickstart full-local             # Kitchen-sink local: all-local-nlp + dedicated workers container + ai-server
make quickstart-help                   # Non-interactive preset reference
make down                              # Stop all services
make service svc=aigateway             # Start the Go AI Gateway data plane on :5563
make help                              # Full target list including boxd workflows
```

The preset-picker writes `langwatch/.env.dev-up` listing only the URLs to override; everything else comes from your `langwatch/.env`. **Credentials never go in the overlay** — only non-rotating infrastructure shape (bucket / endpoint / region / connection-host). For `dev-storage`, refresh AWS SSO credentials in `.env` first via `bash langwatch/scripts/refresh-dev-s3-env.sh` (the launcher hard-fails without S3_SESSION_TOKEN).

The legacy `make dev` / `make dev-nlp` / `make dev-scenarios` / `make dev-test` / `make dev-full` aliases were removed in #4053. Use the preset names directly. `make dev-up` / `make dev-down` / `make dev-logs` still exist for per-worktree isolated stacks (the `dev-up.sh` use case — separate from `quickstart`).

Stateful services (`langwatch-db-data`, `langwatch-clickhouse-data`, `langwatch-redis-data`) share data across worktrees: sign up once, persist across worktree switches. Only one worktree can have postgres or clickhouse `up` at a time — `quickstart` detects collisions and points at the other compose project. Redis is a singleton on host `:6379`.

For per-PR / per-issue cloud environments via boxd, see `dev/docs/boxd-makefile.md` and `make boxd-help`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langwatch/langwatch](https://github.com/langwatch/langwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
