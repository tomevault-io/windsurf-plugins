---
trigger: always_on
description: [中文](./AGENTS-CN.md) | English
---

# Agents.md

[中文](./AGENTS-CN.md) | English

## Project Overview

Sparo OS is an Agentic OS for building and running intelligent apps. The first-class product surfaces are the Tauri desktop app and the CLI, both backed by shared Rust core services. The React/TypeScript Web UI is the desktop app surface.

Focus routine development on:

- `src/apps/desktop` - Tauri 2 desktop shell, commands, capabilities, and desktop-only integration.
- `src/apps/cli` - CLI command surface, terminal/TUI rendering, and CLI-only integration.
- `src/web-ui` - React 18 + TypeScript UI used by the desktop app.
- `src/crates/core` - platform-agnostic business logic, agent runtime, services, storage, paths, and tools.
- `src/crates/events` - platform-agnostic event contracts.
- `src/crates/transport` - adapters between core/events and app surfaces.

Do not describe or design features around a general-purpose app server target unless the user explicitly asks for it. The supported product paths are desktop + Web UI and CLI; Remote Connect relay is a separate infrastructure crate, not a Sparo app server.

## Current Architecture

- `src/crates/core/src/agentic` - agents, prompts, sessions, dialog turns, model rounds, and tool execution.
- `src/crates/core/src/command` - host-agnostic command services shared by desktop and CLI adapters.
- `src/crates/core/src/runtime` - shared process and agentic runtime builders used by desktop and CLI.
- `src/crates/core/src/service` - workspace, config, filesystem, terminal, git, and related services.
- `src/crates/core/src/infrastructure` - AI adapters, app paths, logging, storage, debug ingest, and events.
- `src/web-ui/src/app` - application shell and desktop panels.
- `src/web-ui/src/flow_chat` - chat UI, tool cards, streaming/tool event presentation.
- `src/web-ui/src/tools` - feature tools such as editor, terminal, git, mermaid, and design canvas.
- `src/web-ui/src/infrastructure` - theme, i18n, config, API adapters, and state wiring.
- `src/web-ui/src/design-system` - reusable UI APIs, visual contracts, preview coverage, and AI-facing UI rules.
- `src/web-ui/src/shared` - shared frontend services, markdown rendering, utilities, and types.
- `src/web-ui/src/locales` - translations.

## Development Commands

Use `pnpm` from the repository root.

```bash
pnpm install               # install dependencies
pnpm run desktop:dev       # run the desktop app in development
pnpm run dev:web           # run only the Web UI with Vite
pnpm run type-check:web    # TypeScript check
pnpm run lint:web          # frontend lint
pnpm run build:web         # type-check + Web UI build + Monaco asset verification
pnpm run check:i18n        # locale file/key consistency
pnpm run check:design-system # design-system architecture and styling gate
pnpm run preview:design-system # run the design-system preview app
pnpm run build:design-system   # build the design-system preview output
pnpm run desktop:build     # desktop production build
pnpm run cli:dev -- --help # run the CLI in development
pnpm run cli:build         # build the CLI release binary
pnpm run cli:check         # Rust check for the CLI crate
pnpm run e2e:test          # WebDriverIO E2E suite in debug app mode
```

For Rust-only changes, run the narrowest useful `cargo check` or `cargo test` command for the crate touched.

## Fast Verification Loop

Use E2E as a focused end-to-end feedback loop for large or complex work: new product flows, multi-surface UI features, desktop/Web UI integration, or bugs that have already resisted one or two simpler fixes. In those cases, write or update one small spec that exercises the exact workflow being implemented or repaired, then iterate against that spec until the behavior is correct:

```bash
pnpm run e2e:test:spec -- tests/e2e/specs/<feature>.spec.ts
pnpm run e2e:test:spec:dev -- tests/e2e/specs/<feature>.spec.ts # when Tauri dev/watch should rebuild Rust
```

Keep the E2E proportional: small copy/style/type-only edits usually need only the cheapest relevant check, such as `pnpm run check:web:fast`, `pnpm run type-check:web`, or the narrowest `cargo check`/`cargo test`. For E2E-critical UI controls, add stable `data-testid` hooks. If the focused spec exposes stale helper infrastructure, repair the helper/spec so the e2e validation remains trustworthy.

Use broader suites such as `pnpm run e2e:test:l0`, `pnpm run e2e:test:l0:all`, `pnpm run e2e:test:l1`, or `pnpm run e2e:test` when the change spans their surface area or before a release-style handoff.

## Critical Rules

### Platform Boundaries

Core code must stay platform agnostic.

- In `src/crates/core`, do not depend on Tauri types such as `tauri::AppHandle`.
- Prefer `bitfun_events::EventEmitter`, service traits, and constructor-injected dependencies.
- Desktop-specific code belongs under `src/apps/desktop` or an adapter layer.
- Keep Tauri command DTOs and shared command request/response structs structured and serializable.

### Tauri Commands

Command names are `snake_case` in Rust and invoked as `camelCase` through TypeScript helpers when exposed to the UI.

Always prefer a structured request object:

```rust
#[tauri::command]
pub async fn your_command(
    state: State<'_, AppState>,
    request: YourRequest,
) -> Result<YourResponse, String>
```

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GCWing/Sparo-Agentic-OS](https://github.com/GCWing/Sparo-Agentic-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
