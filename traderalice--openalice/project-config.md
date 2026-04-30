---
trigger: always_on
description: File-driven AI trading agent. All state (sessions, config, logs) stored as files — no database.
---

# OpenAlice

File-driven AI trading agent. All state (sessions, config, logs) stored as files — no database.

## Quick Start

```bash
pnpm install
pnpm dev        # Dev mode (tsx watch, port 3002)
pnpm build      # Production build (backend + UI)
pnpm test       # Vitest
pnpm test:e2e   # e2e test
```

### Pre-commit Verification

Always run these checks before committing:

```bash
npx tsc --noEmit   # Type check (catches errors pnpm build misses)
pnpm test           # Unit tests
```

`pnpm build` uses tsup which is lenient — `tsc --noEmit` catches strict type errors that tsup ignores.

## Working with TODO.md

`TODO.md` at the repo root is the running backlog — deferred work, known
bugs, security gaps, and design items sitting in the on-deck circle.
Unfinished items there compound over time if they're forgotten.

- **Before starting non-trivial work**, scan `TODO.md` for related entries.
  If there's one, either (a) handle it as part of the current change, or
  (b) confirm with the user why you're skipping it so it doesn't drift.
- **When finishing a change**, if it resolves a TODO entry, delete that
  entry in the same commit (git log is the history — the file is a
  future-looking list, not an audit trail).
- **When a new item surfaces mid-work** — a known-broken behaviour you
  don't have scope to fix, a security concern, a half-done UI surface —
  add it with enough context (symptom + suspected location) that the
  next person can start without re-derivation.

## Project Structure

```
src/
├── main.ts                    # Composition root
├── core/
│   ├── agent-center.ts        # Top-level AI orchestration, owns GenerateRouter
│   ├── ai-provider-manager.ts # GenerateRouter + StreamableResult + AskOptions
│   ├── tool-center.ts         # Centralized tool registry (Vercel + MCP export)
│   ├── session.ts             # JSONL session store
│   ├── compaction.ts          # Auto-summarize long context windows
│   ├── config.ts              # Zod-validated config loader (generic account schema with brokerConfig)
│   ├── ai-config.ts           # Runtime AI provider selection
│   ├── event-log.ts           # Append-only JSONL event log
│   ├── connector-center.ts    # ConnectorCenter — push delivery + last-interacted tracking
│   ├── async-channel.ts       # AsyncChannel for streaming provider events to SSE
│   ├── model-factory.ts       # Model instance factory for Vercel AI SDK
│   ├── media.ts               # MediaAttachment extraction
│   ├── media-store.ts         # Media file persistence
│   └── types.ts               # Plugin, EngineContext interfaces
├── ai-providers/
│   ├── vercel-ai-sdk/         # Vercel AI SDK ToolLoopAgent
│   └── agent-sdk/             # Claude backend (@anthropic-ai/claude-agent-sdk, supports OAuth + API key)
├── domain/
│   ├── market-data/           # Structured data layer (typebb in-process + OpenBB API remote)
│   ├── trading/               # Unified multi-account trading, guard pipeline, git-like commits
│   │   ├── account-manager.ts # UTA lifecycle (init, reconnect, enable/disable) + registry
│   │   ├── git-persistence.ts # Git state load/save
│   │   └── brokers/
│   │       ├── registry.ts    # Broker self-registration (configSchema + configFields + fromConfig)
│   │       ├── alpaca/        # Alpaca (US equities)
│   │       ├── ccxt/          # CCXT (100+ crypto exchanges)
│   │       ├── ibkr/          # Interactive Brokers (TWS/Gateway)
│   │       └── mock/          # In-memory test broker
│   ├── analysis/              # Indicators, technical analysis, sandbox
│   ├── news/                  # RSS collector + archive search
│   ├── brain/                 # Cognitive state (memory, emotion)
│   └── thinking/              # Safe expression evaluator
├── tool/                      # AI tool definitions — thin bridge from domain to ToolCenter
│   ├── trading.ts             # Trading tools (delegates to domain/trading)
│   ├── equity.ts              # Equity fundamental tools (uses domain/market-data)
│   ├── market.ts              # Symbol search tools (uses domain/market-data)
│   ├── analysis.ts            # Indicator calculation tools (uses domain/analysis)
│   ├── news.ts                # News archive tools (uses domain/news)
│   ├── brain.ts               # Cognition tools (uses domain/brain)
│   ├── thinking.ts            # Reasoning tools (uses domain/thinking)
│   └── browser.ts             # Browser automation tools (wraps openclaw)
├── connectors/
│   ├── web/                   # Web UI (Hono, SSE streaming, sub-channels)
│   ├── telegram/              # Telegram bot (grammY)
│   └── mcp-ask/               # MCP Ask connector
├── plugins/
│   └── mcp.ts                 # MCP protocol server
├── task/
│   ├── cron/                  # Cron scheduling
│   └── heartbeat/             # Periodic heartbeat
├── skills/                    # Agent skill definitions
└── openclaw/                  # ⚠️ Frozen — DO NOT MODIFY
```

## Key Architecture

### AgentCenter → GenerateRouter → GenerateProvider

Two layers (Engine was removed):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TraderAlice/OpenAlice](https://github.com/TraderAlice/OpenAlice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
