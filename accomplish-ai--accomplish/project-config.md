---
trigger: always_on
description: This file provides general guidance for agents working with code in this repository.
---

# AGENTS.md

This file provides general guidance for agents working with code in this repository.

## Project Overview

Accomplish is an AI automation assistant with a split architecture: `apps/web` contains the standalone React UI, `apps/desktop` is a thin Electron shell that loads web's build output, and `apps/daemon` is a long-lived background process that owns task execution. The daemon spawns `opencode serve` subprocesses and speaks to them via `@opencode-ai/sdk` to run user tasks. API keys are stored with AES-256-GCM encryption.

## Common Commands

```bash
# Development
pnpm dev                                        # Run desktop app in dev mode (web dev server + Electron)
pnpm dev:web                                    # Run web UI only (Vite dev server on localhost:5173)
pnpm dev:clean                                  # Dev mode with CLEAN_START=1 (clears stored data)

# Building
pnpm build                                      # Build all workspaces
pnpm build:web                                  # Build web UI only
pnpm build:desktop                              # Build desktop app (builds web first)

# Type checking & linting
pnpm lint                                       # TypeScript + ESLint (all workspaces)
pnpm typecheck                                  # Type validation (all workspaces)
pnpm lint:eslint                                # ESLint only (flat config)
pnpm format:check                               # Prettier check (no writes)
pnpm format                                     # Prettier write (auto-fix)

# Testing (web workspace — renderer/UI tests)
pnpm -F @accomplish/web test                    # Run all web Vitest tests
pnpm -F @accomplish/web test:unit               # Web unit tests only
pnpm -F @accomplish/web test:integration        # Web integration tests only

# Testing (desktop workspace — main process + preload tests)
pnpm -F @accomplish/desktop test                # Run all desktop Vitest tests
pnpm -F @accomplish/desktop test:unit           # Desktop unit tests only
pnpm -F @accomplish/desktop test:integration    # Desktop integration tests only
pnpm -F @accomplish/desktop test:e2e            # Docker-based E2E tests
pnpm -F @accomplish/desktop test:e2e:native     # Native Playwright E2E tests (serial, Electron requirement)

# Testing (agent-core)
pnpm -F @accomplish_ai/agent-core test          # Run agent-core Vitest tests

# Cleanup
pnpm clean                                      # Clean build outputs and node_modules
```

## Verification After Changes

Always verify before committing. Run the relevant commands for what you changed:

```bash
# After ANY code change — always run typecheck + lint
pnpm typecheck && pnpm lint:eslint && pnpm format:check

# After changing web UI code (components, pages, stores, styles)
pnpm -F @accomplish/web test

# After changing desktop main process or preload code
pnpm -F @accomplish/desktop test

# After changing agent-core code
pnpm -F @accomplish_ai/agent-core test

# Full verification before PR
pnpm lint && pnpm format:check && pnpm -F @accomplish/web test && pnpm -F @accomplish/desktop test && pnpm -F @accomplish_ai/agent-core test
```

## Do NOT

- **Do NOT use `require()`** in agent-core — it is ESM (`"type": "module"`)
- **Do NOT forget `.js` extensions** on imports within agent-core (e.g., `import { foo } from './utils/bar.js'` NOT `./utils/bar`)
- **Do NOT use absolute paths for images** in the web UI — use ES module imports (see Image Assets below)
- **Do NOT modify released migration files** — create a new migration instead
- **Do NOT add root-level test scripts** — tests are workspace-scoped (`-F @accomplish/web`, `-F @accomplish/desktop`, or `-F @accomplish_ai/agent-core`)
- **Do NOT spawn `npx`/`node`** without adding bundled Node.js bin to PATH (see [architecture.md](docs/architecture.md#spawning-npxnode-in-main-process))

## Architecture

See [docs/architecture.md](docs/architecture.md) for full architecture details (monorepo layout, package structure, IPC flow, storage, bundled Node.js).

Key packages:

- `@accomplish_ai/agent-core` — Core business logic, types, storage, MCP tools (ESM, internal workspace package)
- `@accomplish/web` — Standalone React UI (Vite + React Router + Zustand)
- `@accomplish/desktop` — Thin Electron shell (main process + preload), loads web's build output

## Code Conventions

- TypeScript everywhere (no JS for app logic)
- **ESM package**: `@accomplish_ai/agent-core` uses `"type": "module"` — all imports MUST use `.js` extensions
- Shared types go in `packages/agent-core/src/common/types/`
- Core business logic goes in `packages/agent-core/src/`
- UI state via Zustand store actions (in `apps/web/src/client/stores/`)
- IPC handlers in `apps/desktop/src/main/ipc/handlers.ts` must match `window.accomplish` API in preload
- **Always use braces for `if`/`else`/`for`/`while`** - No single-line braceless statements (enforced by `curly` ESLint rule)
- **Avoid nested ternaries** - Use mapper objects or if/else for readability
- **No unnecessary comments** - Don't add comments that restate what the code does. Comments should explain _why_, not _what_
- **Reuse UI components** - Check `apps/web/src/client/components/ui/` before creating new ones


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [accomplish-ai/accomplish](https://github.com/accomplish-ai/accomplish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
