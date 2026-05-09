---
trigger: always_on
description: **Agor** — Multiplayer canvas for orchestrating Claude Code, Codex, and Gemini sessions.
---

# CLAUDE.md

**Agor** — Multiplayer canvas for orchestrating Claude Code, Codex, and Gemini sessions.

Manage git worktrees, track AI conversations, visualize work on spatial boards, and collaborate in real-time.

---

## IMPORTANT: Where the docs live

This file is intentionally high-level. There are three places to look:

1. **The code** — always the ground truth. Open `packages/core/src/types/`, the relevant service in `apps/agor-daemon/src/services/`, or the schema in `packages/core/src/db/schema.{sqlite,postgres}.ts` before assuming behavior.
2. **`apps/agor-docs/pages/guide/`** — user-facing reference pages (also published at [agor.live](https://agor.live)). This is the canonical source for anything users need to configure or understand.
3. **`context/`** — small set of agent-oriented cheat sheets and design docs (file pointers, gotchas, security contracts). Start with [`context/README.md`](context/README.md).

**Rule of thumb:** If a topic has a guide page, read the guide. `context/` is for orientation, not exposition.

---

## Quick Start

**Simplified 2-process workflow:**

```bash
# Terminal 1: Daemon (watches core + daemon, auto-restarts)
cd apps/agor-daemon
pnpm dev

# Terminal 2: UI dev server
cd apps/agor-ui
pnpm dev
```

**IMPORTANT FOR AGENTS:**

- User runs dev environment in watch mode (daemon + UI)
- **DO NOT run `pnpm build`** or compilation commands unless explicitly asked
- **DO NOT start background processes** - user manages these
- Focus on code edits; watch mode handles recompilation automatically

---

## Project Structure

```
agor/
├── apps/
│   ├── agor-daemon/         # FeathersJS backend (REST + WebSocket)
│   ├── agor-cli/            # CLI tool (oclif-based)
│   └── agor-ui/             # React UI (Ant Design + React Flow)
│
├── packages/
│   └── core/                # Shared @agor/core package
│       ├── types/           # TypeScript types (Session, Task, Worktree, etc.)
│       ├── db/              # Drizzle ORM + repositories + schema
│       ├── git/             # Git utils (simple-git only, no subprocess)
│       ├── claude/          # Claude Code session loading utilities
│       └── api/             # FeathersJS client utilities
│
├── apps/agor-docs/         # User-facing docs site (Nextra) — canonical reference
├── context/                 # Agent-oriented cheat sheets and design docs
│   ├── concepts/            # Tight, code-pointer-heavy notes
│   ├── guides/              # Implementation how-tos
│   ├── guidelines/          # House rules (testing, etc.)
│   └── explorations/        # Active design docs referenced from code
│
└── README.md                # Product vision and overview
```

---

## Glossary

Terms you'll see across the codebase, UI, and docs:

| Term | What it is |
|---|---|
| **Worktree** | A first-class git working directory at `~/.agor/worktrees/<repo>/<name>`, on its own branch, with its own dev environment. **Primary card on a board.** Conventionally 1 worktree = 1 feature/PR. |
| **Board** | 2D canvas displaying worktrees as cards. Has zones. |
| **Zone** | Rectangular region on a board with an optional Handlebars **prompt template** that fires when a worktree is dropped in. |
| **Card** | Visual representation of a worktree (or note/markdown) on a board. |
| **Session** | An agent conversation. Required FK to a worktree. Can **fork** (sibling, copies parent context) or **spawn** (child, fresh context window). |
| **Task** | A single user-prompt-and-its-execution within a session. Tasks (not messages) are the queueable unit when a session is busy. |
| **Message** | An individual conversation turn (user / assistant / tool / system) within a task. |
| **Report** | Agent-written markdown summary at task completion. |
| **Environment** | The runtime instance of a worktree's dev server (managed start/stop, ports allocated from `worktree.unique_id`). |
| **Daemon** | The FeathersJS server (`apps/agor-daemon`) that owns the database, services, WebSocket events, and MCP HTTP endpoint. Default port 3030. |
| **Executor** | Process-isolated agent runtime in `packages/executor/`. Spawns Claude / Codex / Gemini / OpenCode via their SDKs. May run as a separate Unix user. |
| **MCP** | Model Context Protocol. Agor exposes itself as an MCP server (`POST /mcp`) so agents can introspect sessions, worktrees, boards, etc. |
| **RBAC** | Worktree-scoped permission tiers (`none`/`view`/`session`/`prompt`/`all`). Feature-flagged via `execution.worktree_rbac`. See "Feature Flags" below. |
| **Unix user mode** | `simple` / `insulated` / `strict` — progressive OS-level isolation tiers. See "Feature Flags" below. |
| **Genealogy** | Parent/child + fork ancestry of a session. Surfaced as a tree inside a worktree card. |
| **Short ID** | First 8 chars of a UUIDv7, used in UI and CLI. Resolved at API boundary via a `resolveShortId` hook. See [`context/concepts/id-management.md`](context/concepts/id-management.md). |
| **Effort** | Reasoning depth knob (`low`/`medium`/`high`/`max`) on `model_config`. Maps to Claude API `output_config.effort`. |

## Where to look first

Tasked with... | Open this
---|---
Mental model | [`context/concepts/core.md`](context/concepts/core.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [preset-io/agor](https://github.com/preset-io/agor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
