---
trigger: always_on
description: **PiBun** — A desktop GUI for the [Pi coding agent](https://github.com/badlogic/pi-mono), built with [Electrobun](https://blackboard.sh/electrobun/docs/).
---

# CLAUDE.md

## Project Identity

**PiBun** — A desktop GUI for the [Pi coding agent](https://github.com/badlogic/pi-mono), built with [Electrobun](https://blackboard.sh/electrobun/docs/).

- **Runtime**: Bun (monorepo with workspaces)
- **Language**: TypeScript (strict)
- **Server**: Bun HTTP + WebSocket — thin bridge to Pi's RPC mode
- **Web**: React 19 + Vite + Zustand + Tailwind CSS v4
- **Desktop**: Electrobun (Bun-native webview, no Chromium)
- **No Effect, no Schema** — plain TypeScript + Zustand everywhere

## Architecture

```
┌─────────────┐     WebSocket      ┌──────────────┐     stdio/JSONL     ┌─────────┐
│  React UI   │ ◄──────────────────►│  Bun Server  │ ◄──────────────────►│ pi --rpc│
│  (Vite)     │                     │              │                     │         │
│  Chat, Tools│                     │ piRpcManager  │                     │ LLM API │
│  Sessions   │                     │ wsServer      │                     │ Tools   │
└─────────────┘                     └──────────────┘                     └─────────┘
         ▲                                  ▲
         └──────── Electrobun webview ──────┘
```

**Core principle: Pi handles state. The server is a thin bridge. Don't reimplement what Pi already does.**

Pi manages: session persistence, model registry, API keys, auto-compaction, auto-retry, tool execution, extensions, skills, prompt templates. The server spawns `pi --mode rpc` subprocesses, pipes JSONL events to WebSocket clients, and translates WebSocket requests into Pi RPC commands. That's it.

## Agent-First Architecture

**The system that builds the thing should shape the thing.**

This codebase is maintained by AI coding agents, not humans with IDEs. Every structural decision optimizes for how agents work: tool calls to read files, exact-text edits to change them, and context windows to reason about them.

### Deep Modules Over Small Files

Humans navigate with Cmd+P and jump-to-definition — file granularity is free. Agents navigate with tool calls — every file read costs time, every cross-file edit is a failure point. So we use **deep modules**: fewer files, richer interfaces, self-contained units.

**Rules:**
- Merge files that always change together (e.g., 5 related store slices → 1 slice per domain)
- Merge files that are always read together (e.g., Pi types + events + commands → one `piProtocol.ts`)
- Merge files that add no abstraction (a 20-line slice with 3 setters is organizational, not architectural)
- Leave files that are already deep (570-line subprocess manager = don't touch)
- Don't create one-file-per-function modules
- Don't split a domain across files unless a file exceeds ~600 lines with distinct concerns

**Metrics that matter:**
- **Tool call count** — how many reads to understand one feature? (target: 3-4, not 7-8)
- **Co-change set size** — how many files must be edited for one new feature? (target: 3-4)
- **Self-containment** — can an agent read one file and fully understand that domain?

### Context Documents

Only 3 mandatory files at session start — not 7, not 12:
- **CLAUDE.md** (this file) — everything an agent needs to orient
- **CONVENTIONS.md** — build rules not expressed in code
- **TENSIONS.md** — living friction log

All decisions, gotchas, reference guides, personality, and human context live HERE, in one file. No cross-referencing web of agent identity files. One read, full context.

### Documentation Lives in Code

Protocol specs live as TSDoc in the type files, not in separate markdown docs. When `wsProtocol.ts` has comprehensive TSDoc, a separate `WS_PROTOCOL.md` becomes stale. The types ARE the spec.

Docs that remain are for things code can't express: high-level architecture (`ARCHITECTURE.md`), operational procedures (`CODE_SIGNING.md`), desktop integration details (`DESKTOP.md`).

## Monorepo Structure

```
pibun/
├── CLAUDE.md                # ← You are here
├── .agents/                 # CONVENTIONS.md, TENSIONS.md
├── .plan/                   # PLAN.md, MEMORY.md, DRIFT.md, SESSION-LOG.md
├── docs/                    # ARCHITECTURE.md, DESKTOP.md, CODE_SIGNING.md, ROADMAP.md
├── reference/               # Read-only reference repos (not committed)
│   ├── pi-mono/             # Authoritative Pi source
│   ├── t3code/              # WebSocket patterns, UI structure
│   └── electrobun/          # Native app patterns
├── apps/
│   ├── server/              # Bun server — Pi RPC bridge + WebSocket
│   ├── web/                 # React/Vite chat UI
│   └── desktop/             # Electrobun native wrapper
└── packages/
    ├── contracts/           # Shared TypeScript types (no runtime logic)
    └── shared/              # Shared runtime utilities
```

## Commands

```bash
bun install                  # install all workspace deps
bun run build                # build all packages
bun run dev:server           # server only (port 24242)
bun run dev:web              # Vite dev server only (port 24269)
bun run dev:desktop          # Electrobun dev mode
bun run build:desktop        # production desktop build (unsigned)
bun run build:desktop:signed # signed + notarized macOS build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khairold/pibun](https://github.com/khairold/pibun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
