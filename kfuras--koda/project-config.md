---
trigger: always_on
description: This file is for AI coding assistants (Cursor, Claude Code, Copilot, and
---

# AGENTS.md — Rules for AI coding assistants working on Koda

This file is for AI coding assistants (Cursor, Claude Code, Copilot, and
similar tools) editing the Koda codebase. It captures architectural
decisions, conventions, and operational rules that would otherwise require
re-learning on every session.

**This file is NOT for the running Koda daemon itself** — that's what
`~/.koda/soul.md` is for. Koda's runtime personality lives in user state;
this file lives in the code repo and describes how to write/modify the
code.

## What Koda is

Koda is a TypeScript daemon built on Anthropic's Claude Agent SDK. It runs
as a persistent background process (via pm2) that handles autonomous
content production for one solo operator. The operator controls it through
Discord; Koda executes tasks via scheduled cron jobs, webhooks, and MCP
servers.

Koda is **explicitly single-tenant**. It is not a multi-user product, not a
SaaS, and should not be refactored toward either without a deliberate
decision. The architecture assumes one human, one `~/.koda/` directory,
one set of OAuth tokens.

## Architecture (read before editing anything non-trivial)

### Entry points

- `src/index.ts` — process entry point (daemon mode). Wires the agent, Discord
  bot, scheduler, webhook server, voice support, and teleport.
- `src/koda.ts` — CLI binary entry point (`#!/usr/bin/env node` shebang).
  Subcommand router. Lives at `dist/koda.js` after build, symlinked globally
  via `npm link`.
- `src/cli.ts` — legacy interactive REPL. Untouched. `npm run cli` still
  works for anyone who wants it. Don't delete without replacing the npm
  script.

### Core modules

- `src/agent.ts` — the persistent streaming `query()` session + isolated
  task runner (`runIsolatedTask`). The persistent session handles Discord
  messages and teleport context. Isolated tasks run with fresh context for
  scheduled jobs and ticks so they don't bloat the main session.
- `src/bot.ts` — Discord bot. Message routing, rate limiting, approval
  reactions, startup message (via `DISCORD_PROACTIVE_CHANNEL`), attachment
  handling, long-message chunking.
- `src/scheduler.ts` — cron-based task runner, tick loop (Sonnet isolated
  sessions), daily digest, auto-backup, missed-task recovery, self-healing.
- `src/dream.ts` — nightly memory consolidation cycle (03:07 daily, LLM-driven).
- `src/runtime.ts` — shared runtime helpers (content-type classification,
  memory freshness checks, observation writes).
- `src/patterns.ts` — reusable behavioral patterns: circuit breaker, state
  file queue, session registry.
- `src/skills.ts` — loads skills from `~/.koda/skills/` into the system prompt.
  Supports both flat `.md` files and directory-based `SKILL.md` format
  (AgentSkills / ClawHub compatible).

### Commands (CLI)

Each CLI subcommand lives in `src/commands/<name>.ts`, exports:
- `description: string`
- `runX(args: string[]): Promise<number>` (exit code)

Commands are registered in `src/koda.ts` → `COMMANDS` map. To add a new
command: create the file, export the two things, add a line to the map.
That's it.

Current commands: `init`, `status`, `update`, `logs`, `restart`, `doctor`,
`skills`, `health`.

### Tools (MCP)

Koda exposes custom tools to its own agent sessions via `src/tools/agent-tools.ts`.
These are Agent SDK MCP tools (NOT external MCP servers). They show up
in the agent's tool list as `mcp__agent-tools__<toolName>`.

External MCP servers (x-mcp, bluesky-mcp, youtube, gmail, etc.) are
loaded from `~/.koda/mcp-servers.json` at runtime via `getMcpServers()`
in `src/agent.ts`.

### Memory pipeline (Layer numbers)

Koda implements a 6-layer memory consolidation pipeline. **Do not break
the layer boundaries.** The design is documented in full at
`docs/memory-architecture.md`. Short version:

| Layer | Location | Producer | Consumer | Cadence |
|---|---|---|---|---|
| 1 — Bootstrap | `~/.koda/soul.md`, `user.md`, `goals.md`, `skills/` | User | Every agent session | Static |
| 2 — Observations | `~/.koda/data/observations.md` | Running agent (via `observe` tool) | Dream cycle | Continuous |
| 3 — Dream cycle | `src/dream.ts` | `dream_cycle` scheduled task | Layers 4 & 5 | 03:07 daily |
| 4 — Daily logs | `~/.koda/data/daily-logs/{date}.md` | `daily_digest` scheduled task | `learnings_review` | 21:00 daily |
| 5 — Learnings | `~/.koda/learnings.md` | `learnings_review` scheduled task | Every agent session | 07:45 daily |
| 6 — Search | `~/.koda/scripts/search-memory.sh` | User or agent | — | On demand |

Each layer has exactly one producer. If you're tempted to add a second
writer to any layer, stop and ask whether you're breaking the invariant.

## Authentication

Koda uses the Claude Agent SDK, which authenticates via one of two paths:

1. **Max subscription** (default): reads the Claude Code CLI's login state
   at `~/.claude/`. User runs `claude login` once, then Koda works
   transparently. No `ANTHROPIC_API_KEY` needed.

2. **API key**: if `ANTHROPIC_API_KEY` is set in `~/.koda/.env`, the SDK
   uses it instead. Useful for non-interactive servers or when Max quota
   runs out.

Koda is the "Agent SDK" class of runtime, which Anthropic has explicitly
confirmed remains eligible for Max subscription usage (confirmed by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kfuras/koda](https://github.com/kfuras/koda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
