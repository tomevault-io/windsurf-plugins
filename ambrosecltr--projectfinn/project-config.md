---
trigger: always_on
description: **Generated:** 2026-05-04
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-05-04
**Commit:** FINN-56 pattern expansion
**Branch:** pattern-expansion-review

## OVERVIEW

Finn is a single-user AI companion that lives in iMessage. Built as a Bun/TypeScript monorepo with a multi-agent architecture: a hot-path agent handles real-time messages, background workers tackle long tasks, and Pattern workers handle scheduled or trigger-driven automations. Runtime state is backed by Postgres (Drizzle ORM) plus Finn's inbuilt profile, conversation, file, worker, connector, and Pattern systems.

## STRUCTURE

```
FinnAI/                        # Git + workspace root
├── AGENTS.md                  # Repo-wide engineering notes for coding agents
├── identity/                  # Personality/voice prompts (FINN.xml, FINN.kids.xml) — runtime injected
├── prompts/                   # Agent process instructions (hot-path, worker, pattern workers, compactor) — first-class runtime artifacts
├── docs/                      # Operational docs (architecture, config, agents, memory, tools, etc.)
├── docker/                    # entrypoint.sh + sandbox.Dockerfile — app boot and worker sandbox image
├── packages/
│   ├── core/                  # Shared types, Zod config, EventBus, logger, errors, tracing
│   ├── db/                    # Drizzle schema + Postgres client (singleton)
│   ├── llm/                   # Provider-agnostic LLM abstraction (Anthropic/OpenAI/Fireworks/DeepSeek/OpenAI-compatible)
│   ├── agents/                # Agent implementations (hot-path, worker, compactor) ← has AGENTS.md
│   ├── tools/                 # Tool definitions for hot-path + worker agents ← has AGENTS.md
│   ├── messaging/             # Spectrum adapter, message routing, sender
│   ├── media/                 # Deepgram STT, ElevenLabs TTS, file storage, attachment processing
│   ├── cron/                  # Shared scheduling helpers
│   ├── patterns/              # Pattern store + scheduler for scheduled/composio automations and run history
│   ├── integrations/          # External clients: Exa/Parallel web, Fal, Composio, BrowserUse, MCP manager
│   ├── toolsets/              # Project-local toolsets with model instructions + allowlisted executors
│   ├── puter/                 # Tauri macOS menu bar app for paired local iMessage/Notes Personal Intelligence
│   ├── web/                   # Vite/React companion app for login, profile, connectors, patterns, and recent pattern runs
│   └── server/                # Hono HTTP server, routes, event wiring — app entry point
├── drizzle.config.ts
├── package.json               # Bun workspace monorepo root
└── tsconfig.json              # Strict TS, ESNext, bundler resolution, @finn/* path aliases
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add/modify hot-path tools | `packages/tools/src/hot-path/` | See tools/AGENTS.md |
| Add/modify worker tools | `packages/tools/src/worker/` | See tools/AGENTS.md |
| Add/modify document or attachment extraction | `packages/media/src/document-extractor.ts` + `packages/media/src/attachment-processor.ts` + `packages/toolsets/src/toolsets/files/` | Keep extraction local-only, bounded, and gated by runtime |
| Change agent behavior | `packages/agents/src/` | See agents/AGENTS.md |
| Modify LLM prompts | `prompts/*.xml` | Treat as code — shapes runtime LLM behavior |
| Change prompt assembly / runtime gating | `packages/agents/src/prompt-factory.ts` + `packages/tools/src/worker/factory.ts` | Capability map controls what prompts/tools expose |
| Change personality/voice | `identity/FINN.xml` | Lowercase always, never split replies |
| Add env vars / config | `packages/core/src/config.ts` | Zod schema + env loader; changes propagate everywhere |
| DB schema changes | `packages/db/src/schema.ts` | Then `bun run db:generate && bun run db:migrate` |
| Add HTTP routes | `packages/server/src/routes/` | Wire in `server/src/index.ts` |
| Change web app UI/behavior | `packages/web/src/` | Main app is in `packages/web/src/main.tsx`; patterns/connectors UI and run history live there |
| Change Puter Mac app behavior | `packages/puter/` | Tauri app; update Rust commands and React UI together |
| Change Puter live bridge behavior | `packages/server/src/puter-bridge.ts` + `packages/server/src/personal-intelligence-service.ts` | Puter PI must use live paired Mac commands, not batch uploads |
| Add/modify project-local toolsets | `packages/toolsets/src/` | Toolset instructions, schemas, and executors used by internal automation like Puter PI |
| Event wiring (worker delivery / Pattern surfacing) | `packages/server/src/event-wiring.ts` | Controls which events persist + deliver to hot-path |
| Memory activity feed events | `packages/server/src/activity-feed.ts` + `packages/integrations/src/memory.ts` | Provider-neutral operational events retained through memory |
| External integrations | `packages/integrations/src/` | MCP, Exa/Parallel web, Fal, Composio, BrowserUse |
| Startup / DI wiring | `packages/server/src/index.ts` | 367-line bootstrap — creates all subsystems |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ambrosecltr/ProjectFinn](https://github.com/ambrosecltr/ProjectFinn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
