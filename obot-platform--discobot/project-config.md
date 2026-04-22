---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# CLAUDE.md

This file provides guidance to coding agents working in this repository.

## Project Overview

Discobot is a coding agent manager. It runs, monitors, and manages its own built-in coding agent across isolated sandboxed sessions. Each session gets its own container with a copy-on-write filesystem, MITM proxy, and agent API. Discobot currently supports Anthropic and OpenAI models, with more model providers to come.

## Commands

### Development

```bash
pnpm dev                # Start all services (backend + Tauri app)
pnpm dev:backend        # Backend only (Svelte UI + Go server + agent watcher)
pnpm dev:app            # Tauri desktop app only
pnpm dev:frontend       # Active frontend only (Svelte UI on port 3100)
pnpm dev:server         # Go backend with hot-reload via air (port 3001)
```

### Build

```bash
pnpm build              # Build server + active frontend + Tauri app
pnpm build:server       # Build Go server binary
```

### Lint & Format

```bash
pnpm check              # Run active frontend checks, backend checks, and shellcheck
pnpm check:fix          # Run UI Prettier + ESLint fixes, backend autofixes, and shellcheck
pnpm check:frontend     # Delegate to the Svelte UI's Prettier + ESLint + typecheck flow
pnpm check:backend      # golangci-lint (server + proxy + agent-go + authservice)
pnpm format             # Run the Svelte UI Prettier formatter
```

### Tests

```bash
pnpm test               # All unit + integration tests
pnpm test:unit          # All unit tests (server, proxy, agent-go, watcher, ui)
pnpm test:ui            # Svelte UI tests only

# Go tests
pnpm test:server        # All server tests
pnpm test:server:unit   # Server unit tests (excludes integration/)
pnpm test:server:integration  # Server integration tests
pnpm test:proxy         # All proxy tests
pnpm test:agent-go      # Agent Go tests

# Single Go test
cd server && go test -v -run TestName ./internal/path/...

# Single Svelte UI test
node --import tsx --test ui/src/lib/components/test/<test-file>.test.ts
```

### CI

```bash
pnpm ci                 # Full CI pipeline: check → test:unit → build
```

## Architecture

### Components

| Component | Language | Port | Purpose |
|-----------|----------|------|---------|
| Frontend | TypeScript (Svelte + Vite) | 3100 | Active web UI |
| Server | Go (Chi + GORM) | 3001 | REST API, session orchestration, container management |
| Agent | Go | — | Container PID 1 init process (workspace setup, AgentFS mount) |
| Agent API | Go | 3002 | Per-container API that drives the AI CLI, SSE streaming |
| Proxy | Go | 17080/17081 | Per-container MITM proxy (auth header injection, Docker registry caching) |

### Data Flow

```
Frontend → REST API (/api/projects/{projectId}/...) → Go Server
                                                        ↓
                                              Docker/VM Container
                                              ├── Agent (PID 1 init)
                                              ├── Agent API (chat/SSE)
                                              └── Proxy (MITM + cache)
```

### Backend Layers

```
Handler (HTTP) → Service (Business Logic) → Store (Data Access) → GORM (SQLite/PostgreSQL)
```

### Resource Hierarchy

```
Project → Workspace (git repo or local folder) → Session (chat thread + container) → Messages + Files
       → Agent (AI config: type, prompt, MCP servers, mode, model)
       → Credential (encrypted API keys / OAuth tokens)
```

### Frontend Patterns

- `./ui` is the active frontend. All frontend work goes here.
- **Styling**: Tailwind CSS v4 with CSS custom properties. Use design tokens (`bg-background`, `text-foreground`, `border-border`) and IDE tokens (`bg-tree-hover`, `bg-diff-add`)
- **Icons**: Theme-aware via `IconRenderer` component. SVGs with `currentColor` must be inlined, not `<img>`

### Svelte UI (`./ui`)

The Svelte UI is the active frontend. Build and develop it from the `./ui` directory:

```bash
cd ui && pnpm build       # Production build
cd ui && pnpm dev         # Dev server (port 3100)
cd ui && pnpm typecheck   # Type-check (svelte-check)
```

#### Component folders

Components live under `ui/src/lib/components/` in three folders, each with a distinct role:

| Folder | Role | Context |
|--------|------|---------|
| `ui/` | Pure primitives — buttons, inputs, dialogs, etc. | None |
| `ai/` | Self-contained compound components | Component-local only |
| `app/` | App shell — session UI, composer, panels | Global app/session/thread contexts |

**`ui/` is always pure.** Never add context consumers here.

**`ai/` is self-contained.** Each subdirectory is a compound component group with its own `context.ts`. The root component (e.g. `AudioPlayer.svelte`) sets local context; children (e.g. `AudioPlayerPlayButton.svelte`) consume it. These components never use the global app/session/thread contexts.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obot-platform/discobot](https://github.com/obot-platform/discobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
