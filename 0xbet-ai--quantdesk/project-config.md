---
trigger: always_on
description: QuantDesk is an AI-agent workspace for quantitative trading.
---

# CLAUDE.md

## Purpose

QuantDesk is an AI-agent workspace for quantitative trading.
Users research, backtest, and validate strategies through async interaction with AI agents (Analyst, Risk Manager).

- **Strategy Desk**: Workspace with budget (USD), target return, and stop-loss constraints. Pick from a curated catalog or generate a custom strategy from natural language.
- **Experiments & Runs**: Organize work into experiments (research threads) within a desk. Each experiment tracks multiple backtest runs with normalized results for comparison.
- **Dataset Management**: Reusable market data scoped per desk — exchange, pairs, timeframe, and date range. Shared across runs.
- **Code Versioning**: Per-desk git workspace. Agent commits strategy code on every change; each run links to its exact commit hash.
- **Paper Trading**: User approves a validated strategy to start paper trading. Engine runs the strategy in paper mode.
- **Engine Adapters**: Pluggable engines for backtesting and paper trading. See `doc/engine/README.md`.
- **Agent Layer**: AI CLI subprocess with session persistence and hipocampus-inspired memory compaction.

## Read This First

- `doc/OVERVIEW.md` — tech stack, repo map
- `doc/agent/TURN.md` — how a single agent turn is executed (CLI subprocess, prompt, session)
- `doc/agent/PAPER_LIFECYCLE.md` — long-running paper trading state machine, observer turns, reconcile
- `doc/agent/MCP.md` — MCP tool glossary (the agent↔server protocol)
- `doc/agent/ROLES.md` — Analyst, Risk Manager, interaction pattern
- `doc/agent/MEMORY.md` — hipocampus-inspired long-term context
- `doc/engine/README.md` — pluggable engine adapter interface (incl. per-engine workspace layout)
- `doc/desk/STORAGE.md` — where a desk's state lives on disk and in the database
- `doc/plans/` — gaps between current code and spec (the only directory where hedging language is allowed)
- `doc/REFERENCES.md` — upstream references (Paperclip, Hipocampus, engine projects, etc.)

## Dev Setup

Prerequisites: Node.js 20+, pnpm 9.15+, Docker (for engine executors only), Claude CLI (`claude`) or Codex CLI (`codex`).

```bash
pnpm install
pnpm dev
```

PostgreSQL runs in-process via `embedded-postgres` (data under `~/.quantdesk/pgdata`) — no Docker required for the database. Docker is reserved for engine executor containers spawned at runtime.

To point at an external Postgres instead, set `DATABASE_URL` before running any script (`dev`, `db:migrate`, `db:seed`, `db:reset`).

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start server + UI in dev mode |
| `pnpm build` | Build all packages |
| `pnpm typecheck` | TypeScript type checking |
| `pnpm check` | Biome linter + formatter |
| `pnpm test` | Vitest test suite |
| `pnpm db:migrate` | Run Drizzle migrations |
| `pnpm db:generate` | Generate migration from schema changes |

## Environment Variables

| Variable | Default |
|----------|---------|
| `DATABASE_URL` | `postgresql://quantdesk:quantdesk@localhost:5432/quantdesk` |
| `PORT` | `3000` |
| `AGENT_MODEL` | `claude-opus-4-6` |
| `LOG_LEVEL` | `info` |
| `QUANTDESK_DEPLOYMENT_MODE` | `local_trusted` — set to `authenticated` to require email/password login |
| `BETTER_AUTH_SECRET` | (auto-generated) — override for production session signing |

## Debugging

Agent session transcripts (tool_call / tool_result / text / thinking chunks)
are persisted per-experiment at `~/.quantdesk/logs/<experimentId>.jsonl`.
Tail the most recent one in real time:

```bash
tail -f "$(ls -t ~/.quantdesk/logs/*.jsonl | head -1)" | jq .
```

This is the fastest way to verify MCP tool calls are actually firing
(`type: "tool_call"` with `name: "mcp__quantdesk__..."`) and to read the
returned tool_result payload.

## Rules

1. **English only** — all code, comments, UI strings, docs, commits. Exception: `README.md` files must be provided in all supported UI languages (`en`, `ko`, `ja`, `zh`, `es`, `pt-BR`, `fr`) using the naming convention `README.<lang>.md` (e.g. `README.ko.md`). The root `README.md` is always English. **Any change to `README.md` must be applied to all language variants in the same commit.**
2. **File refs** — repo-root relative (`src/core/runner.ts:42`), never absolute.
3. **Commits** — `<type>: <description>`. Types: `feat`, `fix`, `refactor`, `docs`, `chore`. **No scoped prefixes** — write `fix: ...`, never `fix(ui): ...` or `feat(server): ...`.
4. **Secrets** — never commit. Use env vars. `.env` is gitignored.
5. **Scope** — backtesting and paper trading only. **Live trading is an explicit forever non-goal** — never implement, design for, or expose real-money trading in APIs or UI. No API keys for trading, no custody, no order routing to real venues.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xbet-ai/QuantDesk](https://github.com/0xbet-ai/QuantDesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
