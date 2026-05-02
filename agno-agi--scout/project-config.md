---
trigger: always_on
description: Scout is an **enterprise context agent** — a single `agno.Agent` with N `ContextProvider`s. Ships with `WebContextProvider`, `WorkspaceContextProvider` (local files via `agno.tools.Workspace`), `DatabaseContextProvider` (the CRM — user's contacts/projects/notes/follow-ups), two `WikiContextProvider`s (a writable `knowledge` wiki and a read-only `voice` wiki), `SlackContextProvider`, `GDriveContextProvider`, and `MCPContextProvider` (any MCP server → one `query_mcp_<slug>` tool on Scout). GitHub,
---

# AGENTS.md

## Project Overview

Scout is an **enterprise context agent** — a single `agno.Agent` with N `ContextProvider`s. Ships with `WebContextProvider`, `WorkspaceContextProvider` (local files via `agno.tools.Workspace`), `DatabaseContextProvider` (the CRM — user's contacts/projects/notes/follow-ups), two `WikiContextProvider`s (a writable `knowledge` wiki and a read-only `voice` wiki), `SlackContextProvider`, `GDriveContextProvider`, and `MCPContextProvider` (any MCP server → one `query_mcp_<slug>` tool on Scout). GitHub, Gmail, and Calendar land in the next release (were built and verified on the `feat/slack-interface` branch; dropped from the ship slice until we can test end-to-end with real tokens).

## Architecture

```
Scout (single Agent — one LLM hop per turn)
  tools = <query_|update_ tools from every registered ContextProvider> + list_contexts
```

Every source is a `ContextProvider`. The database is a provider too: `DatabaseContextProvider` exposes `query_crm` (reads) + `update_crm` (writes), each backed by a dedicated sub-agent so the read path never sees the write engine.

## ContextProvider

`agno.context.provider` defines the base. Every external source subclasses `ContextProvider` and implements:

- `query(question) -> Answer` / `aquery(question) -> Answer` — natural-language read
- `status() -> Status` / `astatus() -> Status` — is the source reachable?

Providers that support writes override `aupdate(instruction) -> Answer` (and optionally `update`). The base raises `NotImplementedError`; `_update_tool()` translates that into a readable "<name> is read-only" error so calling agents see a clean failure. Today only `DatabaseContextProvider` overrides it.

`mode` controls how the provider surfaces itself to the calling agent:

| Mode | Exposure |
|---|---|
| `default` | The provider's recommended exposure. Each subclass decides. |
| `agent` | One `query_<id>` tool wrapping a sub-agent. |
| `tools` | The underlying tools directly. |

`model` swaps the model used by the internal sub-agent (when one is built). `instructions()` returns mode-aware usage guidance for the calling agent.

The full type set:
- `Status(ok: bool, detail: str = "")`
- `Document(id, name, uri=None, source=None, snippet=None)` — a piece of content
- `Answer(results: list[Document] = [], text: str | None = None)` — what `query()` returns

## One write surface

| Write surface | Owner | Call |
|---|---|---|
| `scout_*` user-data tables | `DatabaseContextProvider` write sub-agent | `update_crm(instruction)` — SQL DDL + DML, scoped to the `scout` schema (`get_sql_engine()`) |

Everything else reads. The CRM read sub-agent uses `get_readonly_engine()` (PostgreSQL's `default_transaction_read_only`). The scout engine has a `before_cursor_execute` hook that rejects any DDL/DML targeting `public` or `ai` — so even if Scout is tricked into calling `update_crm` with an out-of-schema statement, the engine rejects it.

## Interfaces

Chat surfaces beyond AgentOS's built-in UI. Wired in `app/main.py` and passed to `AgentOS(interfaces=...)`.

| Interface | Trigger | Notes |
|---|---|---|
| Slack | `SLACK_BOT_TOKEN` **and** `SLACK_SIGNING_SECRET` set | Webhook at `/slack/events`; each Slack thread → a persistent session. `resolve_user_identity=True` maps Slack user IDs to names. Setup: [`docs/SLACK_CONNECT.md`](docs/SLACK_CONNECT.md). |

Both Slack env vars must be set for the interface to light up; otherwise `interfaces=[]` and Scout runs headless + on AgentOS. We explicitly pass `token=SLACK_BOT_TOKEN` because agno's default env-var read is `SLACK_TOKEN` — keeping the `_BOT_` name is intentional.

## Structure

```
scout/
├── __init__.py
├── __main__.py                     # CLI: chat | contexts
├── agent.py                        # The single `scout` Agent
├── instructions.py                 # Scout-tuned prompts: SCOUT_INSTRUCTIONS + CRM read/write
├── settings.py                     # Runtime objects: agent_db + default_model() factory
└── contexts.py                     # create/get/update/close_context_providers + list_contexts tool + status row helpers

# The ContextProvider library (base ABC + shipped providers) lives in `agno.context`
# as of agno 2.6 — see `agno.context.{database,gdrive,mcp,slack,web,wiki,workspace}`.

app/
├── main.py                         # AgentOS entry (lifespan wires contexts; Slack interface if env set)
├── router.py                       # /contexts/* endpoints
└── config.yaml

db/
├── session.py                      # get_sql_engine (guarded) / get_readonly_engine / get_postgres_db / create_knowledge
├── url.py                          # DB URL builder
└── tables.py                       # Canonical DDL: scout_contacts / projects / notes / followups

wiki/
├── knowledge/                      # Prose memory Scout files into (gitignored except README)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agno-agi/scout](https://github.com/agno-agi/scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
