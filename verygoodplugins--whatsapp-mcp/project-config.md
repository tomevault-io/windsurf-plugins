---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Cursor, Codex, etc.) and for human contributors using them in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Codex, etc.) and for human contributors using them in this repository.

This file is the single source of truth for "how to contribute here". `CLAUDE.md` exists for tooling that looks for that filename and points to this file.

## Repository

- **Repo:** [`verygoodplugins/whatsapp-mcp`](https://github.com/verygoodplugins/whatsapp-mcp)
- **Origin remote:** always `origin` (this fork). PRs, issues, and `gh` commands target this fork, not the upstream `lharries/whatsapp-mcp`.
- **Default branch:** `main`. All PRs target `main`.
- **Releases:** automated via [release-please](https://github.com/googleapis/release-please) — do **not** hand-edit `CHANGELOG.md` or version numbers.

## Architecture (read first)

Two components, one repo:

```
whatsapp-mcp/
├── whatsapp-bridge/        # Go bridge — talks to WhatsApp Web via whatsmeow
│   ├── main.go             # REST API + event loop
│   ├── webhook.go          # Outgoing webhook for incoming messages
│   └── store/              # SQLite (whatsapp.db, messages.db) + media — gitignored
├── whatsapp-mcp-server/    # Python MCP server — exposes tools to AI clients
│   ├── main.py             # FastMCP tool definitions
│   ├── whatsapp.py         # DB queries + bridge HTTP client
│   └── audio.py            # FFmpeg helpers
└── .github/                # CI, release, security workflows
```

Data flow: AI client → MCP server (Python) → reads SQLite directly **or** calls bridge REST (`http://localhost:8080/api/*` by default; configurable via `WHATSAPP_API_URL` and `WHATSAPP_BRIDGE_PORT`) → bridge (Go) → WhatsApp Web.

Two SQLite databases:

- `whatsapp.db` — owned by whatsmeow (sessions, contacts, LID map). Treat as opaque.
- `messages.db` — owned by the bridge (chats, messages). Schema is ours.

## Scope rules

Before writing code, check [`ROADMAP.md`](./ROADMAP.md). Anything in "out of scope" should be turned into a polite "won't ship" reply, not a PR.

If unsure whether something is in scope, **open an issue first**. Do not open a PR larger than ~300 LOC without prior discussion.

## PR rules

1. **One concern per PR.** A PR titled "feat: X and also fix Y and refactor Z" gets sent back. Split it.
2. **Conventional commits in the title.** `feat:`, `fix:`, `chore:`, `docs:`, `ci:`, `refactor:`, `test:`, `perf:`. Use `!` (`feat!:`) or `BREAKING CHANGE:` in the body for breaking changes.
3. **Reference an issue** for any `feat:` PR (`Closes #N`). Bug fixes don't strictly require an issue but are easier to review with one.
4. **Update docs in the same PR.** README, `CLAUDE.md`/`AGENTS.md`, or inline tool descriptions if you changed user-visible behavior.
5. **Tests.** Add or update tests for any code you touch in `whatsapp-mcp-server/`. The Go bridge has fewer tests today; matching the existing bar is fine, but don't *remove* coverage.
6. **No drive-by formatting.** Don't reformat files you didn't otherwise change. Keep diffs reviewable.
7. **No new top-level dependencies** without justification in the PR description.
8. **Security-sensitive changes** (auth, file paths, network bind, command exec) get extra scrutiny — call them out in the PR body.

## Local commands

```bash
# Go bridge
cd whatsapp-bridge
go run .                    # dev
go build -o whatsapp-bridge && ./whatsapp-bridge   # release-ish
golangci-lint run           # lint
go test ./...               # tests (sparse today)

# Python MCP server
cd whatsapp-mcp-server
uv sync --extra dev
uv run main.py              # dev
uv run pytest -v            # tests
uv run ruff check .         # lint
uv run ruff format .        # format
```

## CI gates

Every PR runs (see `.github/workflows/`). Not every job is blocking today:

**Blocking — must be green to merge:**

- `Python Lint` (`ruff check` + `ruff format --check`)
- `Python Tests` (`pytest`)
- `Go Lint` (`golangci-lint`)
- `Go Build`
- `Version Consistency` (Python pkg version vs `.release-please-manifest.json`)
- `CodeQL Analysis (Python | Go)`

**Informational — runs on every PR but won't fail the build today (`continue-on-error: true`):**

- `Bandit Security Scan`
- `Python Dependency Audit` (`pip-audit`)
- `Go Vulnerability Check` (`govulncheck`)

A failing blocking job is a hard block — fix it or explain in the PR why it's unrelated. For informational scans, investigate findings and either fix them or note in the PR why they're acceptable.

## Environment variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `WHATSAPP_DB_PATH` | `../whatsapp-bridge/store/messages.db` | SQLite path used by the MCP server |
| `WHATSMEOW_DB_PATH` | `../whatsapp-bridge/store/whatsapp.db` | whatsmeow SQLite (LID ↔ phone resolution via `whatsmeow_lid_map`) |
| `WHATSAPP_API_URL` | `http://localhost:8080/api` | Bridge REST endpoint |
| `WHATSAPP_BRIDGE_PORT` | `8080` | Port the bridge binds to |
| `WEBHOOK_URL` | `http://localhost:8769/whatsapp/webhook` | Outgoing webhook for incoming messages (empty = disabled) |
| `FORWARD_SELF` | `false` | Whether self-sent messages are forwarded |

When adding a new env var: document it here, in `README.md`, and in `.env.example`.

## Gotchas (read before editing)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verygoodplugins/whatsapp-mcp](https://github.com/verygoodplugins/whatsapp-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
