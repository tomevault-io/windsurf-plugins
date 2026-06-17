---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Jeriko

Unix-first CLI toolkit for AI agents. TypeScript + Bun runtime, compiled to standalone binary.
Model-agnostic: any AI with exec capability can control the machine.

**For AI agent command reference:** See `AGENT.md` — the system prompt sent to all AI models.
`CLAUDE.md` is for developers working on Jeriko's codebase.

## Build & Dev Commands

```bash
bun install                          # install deps (workspace root)
bun run dev                          # watch mode (bun --watch src/index.ts)
bun run build                        # compile to standalone binary (~66MB)
bun run typecheck                    # tsc --noEmit (strict mode)
```

## Testing

```bash
bun test                             # all tests
bun run test:smoke                   # fast gates (<100ms)
bun run test:unit                    # all unit tests
bun run test:integration             # all integration tests
bun run test:e2e                     # end-to-end tests

# Run a single test file
bun test test/unit/channel-router.test.ts

# Run a subsystem (12 parallel suites in CI)
bun run test:unit:cli                # CLI components, handlers, hooks, UI primitives, flows, boot, themes, keybindings, permission
bun run test:unit:agent              # orchestrator, model system, tool registry, cache, usage, compaction, subagent, instructions
bun run test:unit:billing            # tiers, license, webhooks, store, stripe-events zod schemas
bun run test:unit:channels           # channel router, adapters
bun run test:unit:connectors         # 29 connector types, OAuth, base retry refactor
bun run test:unit:triggers           # cron, webhook, file, email triggers (safeSpawn-wrapped shell actions)
bun run test:unit:relay              # relay client, protocol, connections
bun run test:unit:security           # API auth (HMAC-canonicalized), escape functions, http-retry, secret-file, spawn-safe
bun run test:unit:shared             # config, output, bus, args, DB, env-parse, diagnostics
bun run test:unit:skills             # skill loader, skill tool
bun run test:unit:webdev             # webdev tool, browser scripts
bun run test:unit:streaming          # SSE parsers, socket stream, drivers (usage extraction)

# Integration suites
bun run test:integration:relay       # real Bun relay server + WebSocket
bun run test:integration:commands    # full daemon boot + HTTP API
bun run test:integration:connectors  # connector definitions + health
bun test test/integration/ollama-live.test.ts  # live LLM turn through Ollama
```

Test preload (`test/preload.ts`) forces `chalk.level = 3` for consistent ANSI output across all environments.

## Architecture (4 layers + platform)

```
Layer 4: Relay     → apps/relay/ (Bun, local dev) + apps/relay-worker/ (CF Worker, production)
Layer 3: Daemon    → src/daemon/kernel.ts (~21-step boot) → agent, API, services, storage
Layer 2: CLI       → src/cli/dispatcher.ts → 51 commands, Ink-based interactive REPL (UI v2 design system)
Layer 1: Shared    → src/shared/ (config, types, output, escape, relay-protocol, urls, skills,
                     http-retry, secret-file, spawn-safe, env-parse, diagnostics, version)
Platform: src/platform/ → OS abstraction (darwin/linux/win32) for native features
```

### Entry Points

- `src/index.ts` — routes to CLI dispatcher
- `jeriko` (no args) → interactive chat REPL (`src/cli/chat.tsx`)
- `jeriko <cmd>` → CLI command (`src/cli/dispatcher.ts` → `src/cli/commands/`)
- `jeriko serve` → daemon boot (`src/daemon/kernel.ts`)

### Key Files

| File | Purpose |
|------|---------|
| `src/daemon/kernel.ts` | ~21-step daemon boot (incl. hooks step 6.4, MCP step 6.5, instructions discovery) |
| `src/cli/dispatcher.ts` | Command registry, global flags, fuzzy matching |
| `src/cli/app.tsx` | Root Ink component (useReducer state, callbacks) |
| `src/cli/backend.ts` | Backend interface — daemon IPC vs in-process agent |
| `src/cli/boot/` | UI v2 provider bridges (theme, keybindings, permission) |
| `src/cli/ui/` | Design-system primitives (types, tokens, layout, chrome, motion, data) |
| `src/shared/config.ts` | JerikoConfig schema, loader (defaults → user → project → env) |
| `src/shared/output.ts` | ok()/fail() output contract, format switching |
| `src/shared/http-retry.ts` | `withHttpRetry` — exponential backoff + Retry-After + jitter |
| `src/shared/secret-file.ts` | `writeSecretFile` — 0o600 + chmod belt-and-braces |
| `src/shared/spawn-safe.ts` | `safeSpawn` — timeout + signal + stderr cap + SIGTERM→SIGKILL |
| `src/shared/env-parse.ts` | `parseEnvInt` / `parseEnvBool` / `parseEnvString` |
| `src/shared/diagnostics.ts` | `VERSION` + `BUILD_REF` + platform facade |
| `src/daemon/agent/agent.ts` | Main agent loop (runAgent async generator, usage ledger, compaction) |
| `src/daemon/agent/cache/` | Anthropic prompt-cache strategy + decorator + composed builder |
| `src/daemon/agent/usage/` | Cross-provider UsageLedger, USD cost, budget gate |
| `src/daemon/agent/compaction/` | auto + reactive(413) + LLM summarize + policy |
| `src/daemon/agent/subagent/` | sync / async / fork / worktree spawn modes + store + notification |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EtheonAI/jeriko-cli](https://github.com/EtheonAI/jeriko-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
