---
trigger: always_on
description: CortexPrism is a self-hosted, open-source AI agent operating system — an autonomous agent runtime
---

# CortexPrism — AI Agent Operating System

## Project Identity

CortexPrism is a self-hosted, open-source AI agent operating system — an autonomous agent runtime
that turns any LLM into a capable digital agent. It provides persistent memory, a rich tool
ecosystem, sandboxed code execution, multi-agent orchestration, a full-featured web UI, and
enterprise-grade security.

- **License**: Apache 2.0
- **Version**: 0.53.0 (see `deno.json`)
- **Repository**: `CortexPrism/cortex` on GitHub
- **CI**: `.github/workflows/ci.yml` (runs on push to `main`)

## Tech Stack

| Layer           | Technology                                                            |
| --------------- | --------------------------------------------------------------------- |
| Runtime         | Deno 2.x (TypeScript strict mode)                                     |
| Database        | libSQL (SQLite-compatible) via `@libsql/client`                       |
| Testing         | Deno test runner                                                      |
| CLI framework   | `@cliffy/command`                                                     |
| LLM SDKs        | Anthropic, OpenAI, Google Generative AI, AWS Bedrock                  |
| Frontend        | Inline SPA (Tailwind CDN, CodeMirror 6, vanilla JS, 78 modular files) |
| Package manager | Deno import maps (`deno.json`)                                        |

## Build & CI Commands

```bash
deno task check      # Type-check all files
deno task lint       # Lint all files
deno task fmt        # Format all files (auto-fix)
deno task test       # Run all tests (sets --allow-all)
deno run --allow-all src/main.ts <command>  # Run CLI
```

CI runs `deno fmt --check`, `deno lint`, `deno check src/main.ts`, `deno test --allow-all` on
ubuntu, macos, and windows.

## Package Structure (v0.48.6+)

The codebase is organized into 6 coarse packages under `packages/` and a composition root in `src/`:

```
packages/
├── core/           — @cortex/core
│   ├── contracts/  — pure interface definitions (ICortexConfig, IDbClient, etc.)
│   └── src/        — config, db, i18n, utils, plugins
├── gate/           — @cortex/gate
│   ├── contracts/
│   └── src/        — security (policy, vault, supervisor), sandbox, vfs
├── ai/             — @cortex/ai
│   ├── contracts/
│   └── src/        — agent, tools, memory, llm, pipeline, skills
├── server/         — @cortex/server
│   ├── contracts/
│   └── src/        — server, hub, channels, a2a, mcp, voice, workspace, codegraph
├── infra/          — @cortex/infra
│   ├── contracts/
│   └── src/        — processes, services, scheduler, ipc, triggers, workflow, observability
└── cli/            — @cortex/cli
    ├── contracts/
    └── src/        — cli commands, tui

src/
├── agent/          — agent loop orchestrator (81 lines)
│   stages/         — 7 pipeline stages (setup, history, assessment, prompt-builder, model-selector, llm-stream, tool-executor)
│   post/           — post-turn modules (response, background, cleanup)
│   helpers/        — shared helpers (preferences, strip-tool-calls, nanoid)
│   pipeline/       — pipeline context
├── server/
│   server.ts       — HTTP server entry (composition root for server)
│   new-router.ts   — route dispatcher (replaced 6,075-line monolith)
│   routes/         — 69 route modules (one per API area)
│   ui/
│   │   mod.ts      — UI assembler (concatenates JS + HTML)
│   │   js/         — 29 concatenated JS modules
│   │   pages/      — 46 page HTML templates
│   │   shared/     — shared utilities
│   │   css.ts      — embedded CSS
│   │   shell.ts    — sidebar/layout HTML
│   │   providers.ts
├── main.ts         — CLI entry point (composition root)
└── tests/          — 30 test files (flat structure)
```

### Dependency Graph

```
@cortex/core ← @cortex/gate ← @cortex/ai ← @cortex/server ← @cortex/cli
                                    ↖               ↗
                              @cortex/infra
```

- `core` has zero internal dependencies
- `gate` depends only on `core`
- `ai` depends on `core` + `gate`
- `server` depends on `core` + `ai`
- `infra` depends on `core` + `ai`
- `cli` is the composition root, depends on all

### Contracts

Each package defines pure TypeScript interfaces in `packages/<name>/contracts/`. These have zero
runtime dependencies and define the boundaries between packages. Common contracts:

- `ICortexConfig`, `IProviderConfig` (`core/contracts/config.ts`)
- `ITool`, `IToolRegistry`, `IToolContext` (`ai/contracts/tools.ts`)
- `IAgentLoop`, `IAgentTurnOptions` (`ai/contracts/agent.ts`)
- `ILLMProvider`, `ILLMRouter` (`ai/contracts/llm.ts`)
- `IMemoryStore`, `IEpisodicStore` (`ai/contracts/memory.ts`)
- `IPipelineHook`, `IPipelineManager` (`ai/contracts/pipeline.ts`)
- `IPolicyEngine`, `IVault` (`gate/contracts/policy.ts`)
- `ISandboxProvider` (`gate/contracts/sandbox.ts`)
- `IRouteHandler`, `IRouteTable` (`server/contracts/router.ts`)
- `IWSHub`, `IWSHandler` (`server/contracts/websocket.ts`)
- `IScheduler`, `IJobRow` (`infra/contracts/scheduler.ts`)
- `IServiceManager` (`infra/contracts/services.ts`)
- `ICommand`, `ICommandRegistry` (`cli/contracts/commands.ts`)

## Key Architectural Patterns

### Agent Loop (`src/agent/loop.ts`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CortexPrism/cortex](https://github.com/CortexPrism/cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
