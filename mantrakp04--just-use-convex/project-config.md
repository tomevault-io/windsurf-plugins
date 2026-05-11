---
trigger: always_on
description: AI-powered agentic chat platform — multi-tenant, real-time, with org/team support. Features multi-step planning agents with sub-agents, Daytona sandbox code execution (PTY terminals, file ops), vector search (RAG), and tool approval workflows. Cloudflare Workers + Durable Objects for persistent agent state, Convex for reactive backend, TanStack Start for SSR.
---

# AGENTS.md

## Project Overview
AI-powered agentic chat platform — multi-tenant, real-time, with org/team support. Features multi-step planning agents with sub-agents, Daytona sandbox code execution (PTY terminals, file ops), vector search (RAG), and tool approval workflows. Cloudflare Workers + Durable Objects for persistent agent state, Convex for reactive backend, TanStack Start for SSR.
THIS APP HAS NO USERS, thers no real data yet make whatever changes u want dont worry about it we will figure it out when we ship, this is super greenfield, its ok u can change the schema compleatly we are trying to get it in a good shape.

## Tech Stack
| Layer | Stack |
|-------|-------|
| **Runtime** | Bun |
| **Frontend** | React 19, TanStack Start/Router/Query, Tailwind v4, shadcn/ui (base-mira), Jotai, Motion 12, Xterm, Streamdown, Rive |
| **Backend** | Convex 1.31, Convex Ents (relationships), Better Auth 1.4 (org plugin), Convex Helpers (triggers, aggregates) |
| **Agent** | Cloudflare Workers, Alchemy (IaC), Durable Objects (sqlite), VoltAgent Core (planning/sub-agents), OpenRouter, Daytona SDK (sandboxes), Exa (web search), Composio (integrations), Cloudflare Vectorize (RAG) |
| **Build** | Turborepo, Vite 7 |

## Monorepo Structure
```
apps/web/              # TanStack Start frontend (React 19 SSR)
  src/
    components/        # UI — chat/, sandboxes/, todos/, dashboard/, auth/, ai-elements/, ui/
    providers/         # Context providers (agent.tsx — isolated React roots per chat)
    routes/            # File-based routing — (public)/, (protected)/
    store/             # Jotai atoms (chatSettings, favoriteModels, dashboard, sandbox)
    hooks/             # useChats, useSandbox, useAttachments, useOpenrouterModels, usePaginatedQuery
    lib/               # Utilities, motion presets
packages/
  agent/               # Cloudflare Workers agent (Alchemy-managed)
    src/
      agent/           # AgentWorker (AIChatAgent), ConvexAdapter, prompts
      tools/           # web_search, ask_user, sandbox/ (file ops, PTY terminals, code interpreter)
    alchemy.run.ts     # Alchemy IaC — DurableObject, Vectorize, secrets
  backend/             # Convex backend
    convex/
      tables/          # Ent definitions (chats, sandboxes, todos, attachments)
      chats/           # Chat CRUD, search, stats
      sandboxes/       # Sandbox CRUD, Daytona lifecycle triggers
      todos/           # Todo CRUD with member assignment
      lib/             # ConvexAdapter, auth helpers, custom functions (zQuery/zMutation)
  config/              # Shared tsconfig.base.json
  env/                 # T3 Env — exports ./web, ./backend, ./agent
```

## Commands
```bash
bun run dev            # Start everything (Vite + Convex + Alchemy)
bun run dev:web        # Frontend only
bun run dev:server     # Convex backend only
bun run build          # Production build
bun run check-types    # Turborepo type check (MANDATORY)
```

### Agent Commands
```bash
cd packages/agent
bunx alchemy dev alchemy.run.ts      # Local dev
bunx alchemy deploy alchemy.run.ts   # Deploy to Cloudflare
bunx alchemy destroy alchemy.run.ts  # Tear down infrastructure
```

### Type Checking
```
// MANDATORY RUN at the end
bun check-types
```
Always run `bun check-types` after code changes before finalizing. If it fails, re-run until it passes.

### On Finish

Always play `finish.wav` when done working to notify me. This is mandatory before your final response.
Run from repo root and do not skip silently on failure.
```bash
test -f finish.wav && paplay finish.wav
```
If playback fails, explicitly report that in the final response with the command error.

## Communication Style

**Be concise and direct. No fluff. Match the energy.**

User uses casual language ("bro", "dawg", "ugh"). Keep responses terse and actionable. When something breaks, diagnose fast, fix faster.

### Clarification Gate (Mandatory)

- before implementing any non-trivial change, ask for confirmation if scope/intent is not explicitly specified
- do not assume architectural behavior for stateful flows (worker lifecycle, mode switching, persistence, auth propagation, background execution)
- if multiple reasonable implementations exist, present options briefly and wait for selection before coding
- when a change can affect cross-context behavior (chat vs workflow, server vs client, trigger vs interactive path), ask first and get approval
- default to asking one targeted clarifying question rather than executing on inferred intent

---

## DO

- **Infer and derive types from existing packages** — avoid new types; use `Pick`, `Omit`, and built-in TS utilities
- **Check existing patterns** in codebase before implementing
- **Cross-check server/client impact** — if you edit server-side code, verify client usage, and vice versa
- **Use Context7 for third-party SDK API verification** before integrating

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mantrakp04/just-use-convex](https://github.com/mantrakp04/just-use-convex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
