---
trigger: always_on
description: Penny is a **single-player, locally hosted** personal-finance agent: it syncs
---

# Penny

Penny is a **single-player, locally hosted** personal-finance agent: it syncs
bank transactions via Plaid, categorizes them with an LLM against a two-level
taxonomy, and answers natural-language questions about the user's finances —
through a local web UI (`penny serve`) and as a Claude Code plugin
(`penny mcp`), both over the same database and workspace. No accounts, no
tenants: the user's machine and database are the boundary.

Built on two external packages by the same author:

- **[agent-harness](https://github.com/adambossy/agent-harness)** (Python) —
  the agent loop, model providers, sessions, sandboxes, skills, tool
  decorator. Penny never reimplements these.
- **[agent-ui](https://github.com/adambossy/agent-ui)** (React) — chat
  components (`Message`, `Composer`) speaking the Vercel AI SDK UI
  message-stream protocol.

The hosted multi-tenant product (Clerk auth, billing, households, tenant
isolation, Fly/Modal deploy) lives in the private **penny-web** repo, which
consumes this core as a pinned git dependency and composes it via
`create_app` (see "Host composition seam"). The pre-split monolith is
preserved on the long-lived `legacy/saas-monolith` branch — the hosted
product deploys from that freeze until penny-web's integration lands.
Plan of record: `docs/superpowers/specs/2026-07-27-single-player-local-first-design.md`.

## Canonical vs. non-canonical artifacts

Everything checked in is one of two kinds:

- **Canonical** — the current source of truth: backend/frontend code, the
  prompts in `.prompts/`, the taxonomy seed, `REQUIREMENTS.txt`, `plugin/`,
  and these agent docs. If canonical says something false, that is a bug.
- **Non-canonical** — point-in-time records kept for **history, not truth**:
  plans (`plans/`), superseded specs, migration/cut-over scripts, and
  transient one-off tooling (`backend/transient/**` — excluded from the
  ruff/pytest gates, exempt from pattern expectations, deletable once spent).

Directives: canonical wins; keep the history, don't delete it; segregate,
never intermingle; don't maintain non-canonical code.

## Layout

```
backend/
├── penny/
│   ├── api/            # app.py (create_app factory) + routes.py (chat API),
│   │                   #   bridge.py (harness events → AI SDK SSE), main.py
│   │                   #   (default instance), persistence/ (app_* tables:
│   │                   #   conversations, reminders, onboarding)
│   ├── agent_factory.py# builds the Agent: model, prompt render, toolsets
│   ├── mcp_server.py   # `penny mcp` stdio server (toolsets → harnesses)
│   ├── daemon.py       # scheduler loop (sync + weekly report)
│   ├── service_install.py # launchd / systemd-user installation
│   ├── settings.py     # workspace config.toml → env defaults
│   ├── identity.py     # workspace-minted stable local user UUID
│   ├── tools/          # thin @tool wrappers the agent sees
│   │   └── _services/  # service implementations (categorizer, persister, sync…)
│   ├── plugins/amazon/ # self-contained Amazon toolset
│   ├── adapters/       # db (SQLAlchemy façade + models), cache, clients (plaid), storage (R2)
│   ├── taxonomy/, rules/, memory/, services/, eval/, security/, observability/
│   ├── workspace.py    # ~/.penny workspace resolution ($PENNY_WORKSPACE;
│   │                   #   an existing ~/.transactoid is honored)
│   ├── bootstrap.py    # idempotent create_schema + taxonomy seed
│   ├── cli.py          # init / serve / daemon / mcp / sync / run / eval / migrate
│   └── prompts.py      # promptorium-backed load_prompt()
├── .prompts/           # prompt source of truth (promptorium layout)
├── .agent/skills/      # agent-harness SkillRegistry root (8 skills)
├── configs/taxonomy.yaml  # taxonomy seed
├── db/migrations/      # alembic chain (029 = single-player fork point)
├── transient/          # non-canonical one-off tooling (e.g. hosted-data export)
└── scripts/
frontend/               # Vite + React 19 npm workspace:
├── packages/ui         #   @penny/ui (design system: Gallery, Wordmark, …)
├── packages/chat-ui    #   @penny/chat-ui (ChatScreen, drawer, Plaid card…)
└── src/                #   thin single-player app shell consuming both
plugin/                 # Claude Code plugin (manifest, .mcp.json, skills →
                        #   symlinks into backend/.agent/skills)
```

`backend/` is the only Python project (uv workspace of one; root `uv.lock`).

## Dev loop

```bash
# Backend (from repo root) — SQLite default lives at ./backend/penny.db
uv run --project backend uvicorn penny.api.main:app --host 127.0.0.1 --port 8000 --reload

# Frontend (from frontend/) — proxies /api to :8000
npm run dev

# The product, as users run it (serves frontend/dist statically):
uv run --project backend penny serve
```

- **agent-harness is a pinned git dep** (`@v0.2.0`), so `uv sync --frozen`
  installs identically everywhere. To hack on it locally, per-machine:
  `uv sync --frozen && uv pip install -e ~/code/agent-harness` (re-run after
  any `uv sync`).
- **agent-ui** resolves from the published npm package (`@adambossy/agent-ui`
  in `package.json`) **by default**. The `resolve.dedupe` for react stays as a
  guard against a second React copy (which blank-screens the app).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adambossy/penny](https://github.com/adambossy/penny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
