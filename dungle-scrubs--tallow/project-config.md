---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Commands

```bash
# Build (must build tallow-tui fork first)
bun run build

# Build tallow-tui fork only
cd packages/tallow-tui && bun run build

# Watch mode (core only — does NOT watch tallow-tui)
bun run dev

# Typecheck core
bun run typecheck

# Typecheck extensions (separate tsconfig)
bun run typecheck:extensions

# Lint + format
bun run lint
bun run lint:fix

# Tests (bun:test)
bun test                              # all tests
bun test extensions/tasks             # single extension's tests
bun test extensions/__integration__   # integration tests

# E2E: verify all slash commands register
node tests/e2e-commands.mjs

# Run from source
bun dist/cli.js
```

### Pre-commit hook

The husky hook runs `typecheck`, `typecheck:extensions`, and `lint-staged`
(biome check on staged files). All three must pass.

### CLI: `--tools` flag

Restrict which tools are available in a session. Accepts comma-separated
tool names or preset aliases.

**Individual tools:** `read`, `bash`, `edit`, `write`, `grep`, `find`, `ls`

**Presets:**
- `readonly` → `read`, `grep`, `find`, `ls` (no file modifications, no shell)
- `coding` → `read`, `bash`, `edit`, `write` (default set)
- `none` → no tools (chat only)

```bash
tallow --tools readonly          # Safe exploration mode
tallow --tools read,grep,find    # Custom subset
tallow --tools none              # Pure conversation
```

When `--tools` is used, the final active tool set is restricted to the
explicit allowlist.
Extension-registered tools still load, but any tool whose name is not in the
allowlist is deactivated and blocked at runtime.

## Architecture

Tallow is a CLI coding agent built on the
[pi](https://github.com/nicobrinkkemper/pi-coding-agent) framework. The core
is thin — most functionality lives in bundled extensions.

### Core (`src/`)

| File | Role |
|------|------|
| `config.ts` | Identity constants, path resolution, env-var bootstrap. `TALLOW_HOME` resolves via `~/.config/tallow-work-dirs` overrides, falling back to `~/.tallow`. Env vars are set at **module scope** (not in `bootstrap()`) because ESM hoists imports — pi reads them before `bootstrap()` runs. `op://` secrets from `~/.tallow/.env` use a two-phase load: sync plain-value parse in `bootstrap()`, then parallel async `opchain` resolution in `resolveOpSecrets()` (called from `createTallowSession()`). Resolved values are kept in `process.env` only — never persisted to disk. |
| `sdk.ts` | `createTallowSession()` — the main entry point (~2900 lines). Wires auth, models, settings, resource loading, session management, extension discovery, and plugin resolution. Also defines inline factory extensions for: system-prompt rebranding, context-budget planning, tool-result retention/guarding, output truncation detection, project trust management, telemetry instrumentation, image path injection, and explicit tool restrictions. These are extension-shaped but live in core because they need closure over session-scoped state. |
| `cli.ts` | Commander CLI. Parses flags, calls `createTallowSession()`, dispatches to interactive/print/rpc mode. |
| `install.ts` | Interactive installer (clack prompts). Discovers bundled extensions/themes, copies templates to `~/.tallow/`. |
| `index.ts` | SDK public surface — re-exports from config, sdk, and pi framework. |

### Extension loading priority

`sdk.ts` discovers bundled extensions from `extensions/`, then checks
`~/.tallow/extensions/` for user overrides. If a user extension shares a
name with a bundled one, the bundled version is skipped and the user's
version loads instead. This is tracked in `extensionOverrides` and
surfaced to the user on startup.

### Extensions (`extensions/`)

Each extension is a directory with `extension.json` (metadata) and `index.ts`
(default export receiving `ExtensionAPI`). Extensions register tools, commands,
hooks, and event handlers through the pi framework API.

Extensions have a **separate tsconfig** (`tsconfig.extensions.json`) using
`moduleResolution: "bundler"` — different from core's `Node16`. Extension
tests use `bun:test` and live in `__tests__/` subdirectories. Integration
tests that need a full session use test-utils (`test-utils/session-runner.ts`,
`test-utils/mock-model.ts`).

Key patterns:
- Tools: `pi.registerTool({ name, parameters: Type.Object({...}), execute })` — uses `@sinclair/typebox` for schemas
- Commands: `pi.registerCommand("name", { description, handler })` — handler receives `(args, ctx)`
- Events: `pi.on("before_agent_start" | "context" | "turn_start" | ...)` — return mutations or void
- UI: `ctx.ui.notify()`, `ctx.ui.confirm()`, `ctx.ui.input()`, `ctx.ui.custom()`, `ctx.ui.setWorkingMessage()`

### Extension startup policy (lazy `session_start`)

- Keep `session_start` handlers minimal and non-blocking.
- Defer heavy work (network connects, deep filesystem scans, index builds) to first-use triggers like
  `before_agent_start`, `input`, command handlers, or tool execution.
- Use `extensions/_shared/lazy-init.ts` for one-time execution with in-flight dedupe.
- If deferred setup fails, surface a clear `ctx.ui.notify(..., "error")` and keep retries deterministic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dungle-scrubs/tallow](https://github.com/dungle-scrubs/tallow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
