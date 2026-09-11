---
trigger: always_on
description: This project (`supernova`) is a desktop GUI for interacting with a Pi-based coding agent.
---

# agents.md

## Purpose

This project (`supernova`) is a desktop GUI for interacting with a Pi-based coding agent.

The goal is to build a **custom, opinionated user interface** for agent-driven coding workflows without reimplementing the agent itself.

The Pi SDK is used as the execution engine (https://pi.dev/docs). This repository focuses on:

- UI/UX
- interaction model
- feature layer (todos, planning, subagents, etc.)

The goal is for Supernova to be fast, performant and have an ultra-polished user experience.

### Core Priorities

1. Performance first.
2. Reliability first.
3. Keep behavior predictable under load and during failures (session restarts, reconnects, partial streams).

If a tradeoff is required, choose correctness and robustness over short-term convenience.

## High-Level Architecture

```text
apps/server    → Headless Node API + user-facing CLI. Owns Pi runtime, workspace access, and HTTP/WebSocket APIs. Never hosts or bundles the UI.
apps/desktop   → Electron shell. Starts a local API child and loads its own bundled web UI.

packages/web            → Independently hosted React/Vite client bundle. No native/filesystem assumptions.
packages/agent-runtime  → Agent runtime services and provider SDK integrations (Node-only), consumed by the server.
packages/contracts      → Shared Effect schemas, RPC definitions, and domain contracts used by server and web.
```

### Runtime Model

```text
Standalone server:
user terminal → supernova-server → server owns runtime/filesystem/workspaces (no UI)

Browser development:
dev launcher → local API child + Vite UI host → browser connects through the UI host's WebSocket proxy

Desktop app:
Electron app → spawns bundled API on an OS-assigned port → BrowserWindow loads supernova://app and connects to the API endpoint supplied by preload
```

- The **server process** is the authority for native capabilities: Pi runtime, workspace filesystem access, subprocesses, shell/git, credentials, sessions, and API/WebSocket routing.
- The **web package** is a pure client UI. It must communicate with server APIs and must not assume browser-local filesystem/native access.
- The **contracts package** defines shared API/RPC boundaries and serializable domain types. Keep it environment-neutral and free of runtime ownership logic.
- The **desktop app** is a convenience shell and OS integration layer. It owns bundled renderer asset loading, not API web serving or Pi runtime logic.
- Remote/LAN browser access means operations happen on the machine running `apps/server`, not the machine running the browser.

## Tech Stack

```text
Runtime:
- Bun (package manager, scripts, workspace)

Monorepo:
- Turborepo

Frontend:
- React
- TypeScript
- Vite

Desktop:
- Electron

Agent:
- @earendil-works/pi-coding-agent

Architecture:
- Effect (for services / runtime composition)
```

### Rules

- Use **Bun** for all package management and scripts (`bun install`, `bun run`)
- Do not mix npm/yarn/pnpm unless strictly required by external tooling
- Use **workspace packages** (`apps/*`, `packages/*`)
- Prefer **TypeScript everywhere**
- Always run verification commands from the repository root because this is a Turborepo workspace:
  - `bun run test`
  - `bun run typecheck`
  - `bun run lint`
  - `bun run prettier`
- Before editing a file, check whether its package directory contain a nested `AGENTS.md` and read it. Follow those local instructions in addition to this root file for files under that scope.

## Reference Repositories

Additional repositories may be cloned under `.context/` to provide local context. When a user asks how other projects solve a problem, implement a pattern, or structure similar functionality, check `.context/` first before looking elsewhere.

Known useful references include:

- `.context/effect` for Effect examples, internals, and idioms.
- `.context/pi` for Pi SDK and related monorepo patterns.

Other repositories may also be present in `.context/`; inspect the directory when broader examples would help.

## Maintainability

Long term maintainability is a core priority. If you add new functionality, first check if there is shared logic that can be extracted to a separate module. Duplicate logic across multiple files is a code smell and should be avoided. Don't be afraid to change existing code. Don't take shortcuts by just adding local logic to solve a problem.

## Code Standards

- Favor readability over micro-optimizations: straightforward control flow, early returns, and clear naming. Extract constants for magic numbers/strings and keep inline styles minimal (lean on classes or computed style helpers).
- Use `import type` for types; keep strings double-quoted and favor `const` over `let`.
- Use kebab-case for files and folders.
- Read files in full before making wide-ranging changes, before editing files you have not already fully inspected, and when the user asks you to investigate or audit something. Do not rely only on search snippets for broad changes.
- Single-line helper functions with a single call site are forbidden; inline them instead.
- Always ask before removing functionality or code that appears to be intentional.
- Do not preserve backward compatibility unless the user explicitly asks for it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattiacerutti/supernova](https://github.com/mattiacerutti/supernova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
