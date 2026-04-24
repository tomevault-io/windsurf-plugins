---
trigger: always_on
description: Guidelines for agents and developers working in this codebase.
---

# Caspian Development Guide

Guidelines for agents and developers working in this codebase.

## What is Caspian?

Caspian is an Electron desktop app for managing multiple git worktrees and development environments simultaneously. Each "node" represents an isolated workspace (branch or worktree) with its own terminal sessions, file editors, and state.

## Tech Stack

| Layer | Tech |
|-------|------|
| Runtime | Bun |
| Desktop | Electron + React |
| Build | electron-vite |
| State | Zustand |
| IPC | tRPC (trpc-electron) |
| UI | TailwindCSS v4 + shadcn/ui |
| Database | SQLite (better-sqlite3) + Drizzle ORM |
| Linting | Biome |

## Commands

```bash
bun dev           # Start development
bun test          # Run tests
bun run build     # Production build
bun run lint      # Check lint issues
bun run lint:fix  # Auto-fix lint
bun run typecheck # Type check
```

---

## Process Architecture

**The most important constraint in this codebase:**

```
┌─────────────────────────────────────────────────────────────┐
│  MAIN PROCESS (src/main/)                                   │
│  ✓ Node.js modules (fs, path, child_process, etc.)          │
│  ✓ Database access, git operations, file I/O                │
└─────────────────────┬───────────────────────────────────────┘
                      │ tRPC over IPC
┌─────────────────────▼───────────────────────────────────────┐
│  RENDERER PROCESS (src/renderer/)                           │
│  ✗ NO Node.js modules - browser environment only            │
│  ✓ React UI, Zustand stores, user interactions              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  SHARED CODE (src/shared/, src/lib/)                        │
│  ✗ NO Node.js modules - bundled for both processes          │
│  ✓ Types, constants, pure utilities                         │
└─────────────────────────────────────────────────────────────┘
```

**If you import a Node.js module in renderer or shared code, the app will crash at runtime.**

Need Node.js functionality in the UI? Add a tRPC procedure in main and call it from renderer.

---

## Core Concepts

### Repository → Worktree → Node

```
Repository (project)
├── Worktree: main branch (physical directory)
│   └── Node: "main" workspace
├── Worktree: feature-x (physical directory)
│   └── Node: "feature-x" workspace
└── Node: "quick-fix" (branch-only, no worktree)
```

- **Repository**: A git repo the user has opened
- **Worktree**: A git worktree (physical checkout of a branch)
- **Node**: An active workspace in Caspian's UI — can be backed by a worktree or just reference a branch

### Tabs and Panes

Each node has tabs. Each tab has panes (split views). Panes contain terminals or file editors.

```
Node (workspace)
├── Tab 1
│   ├── Pane A (terminal)
│   └── Pane B (file editor)
└── Tab 2
    └── Pane C (terminal)
```

State lives in `src/renderer/stores/tabs/store.ts`.

---

## Project Structure

```
src/
├── main/                     # Main process (Node.js OK)
│   ├── lib/
│   │   ├── terminal/         # Terminal daemon management
│   │   ├── terminal-host/    # PTY daemon client
│   │   ├── node-runtime/     # Runtime abstraction layer
│   │   ├── node-init-manager.ts
│   │   └── local-db/         # SQLite initialization
│   ├── windows/main.ts       # Main window creation
│   └── index.ts              # Entry point
│
├── renderer/                 # Renderer process (browser only)
│   ├── components/           # Shared React components
│   ├── screens/main/         # Main app screen
│   ├── routes/               # TanStack Router pages
│   ├── stores/               # Zustand stores
│   ├── react-query/          # tRPC hook wrappers
│   └── lib/                  # Renderer utilities
│
├── lib/                      # Shared libraries
│   ├── trpc/routers/         # tRPC procedure definitions
│   └── local-db/schema/      # Drizzle schema
│
├── shared/                   # Types and constants (NO Node.js)
│   ├── types/
│   └── constants.ts
│
├── ui/                       # UI component library
│   └── components/ui/        # shadcn components (kebab-case)
│
├── preload/                  # Electron preload scripts
└── resources/                # Static assets, migrations
```

---

## Design Principles

These principles guide architectural decisions. Follow them to keep the codebase maintainable.

### Separation of Concerns

**Separate by ownership and lifecycle.** Transport (routes, IPC handlers), orchestration (tRPC procedures), and domain logic belong in distinct layers. A tRPC procedure should validate input, call domain functions, and return results — not contain business logic itself.

**Co-locate by lifecycle, not by type.** Feature-specific code lives together. Don't split a feature across `/utils`, `/hooks`, `/components` directories — keep related code in one place.

**Boundary layers own error handling.** Domain utilities return data or throw specific errors. Only boundary code (tRPC procedures, React error boundaries) should catch and transform errors for consumers.

### Minimal Coupling

**Keep modules self-contained with narrow public APIs.** Each module should do one thing well and expose only what's necessary.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheCaspianAI/Caspian](https://github.com/TheCaspianAI/Caspian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
