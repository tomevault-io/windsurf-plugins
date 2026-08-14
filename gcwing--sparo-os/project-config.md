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

Use the verification strategy below to choose checks for Rust, Web UI, design-system, locale, and E2E work.

## Verification Strategy

Verification is risk-based. Prefer the cheapest check that gives real confidence for the changed boundary; do not run heavy checks by default.

- Skip automated checks for low-risk docs, comments, prompts, copy, logs, and obvious mechanical edits. Briefly explain when checks are skipped.
- Run formatting only when formatting may be affected, and only after edits have settled.
- Run narrow static checks only when compiler/type feedback materially reduces risk:
  - Rust: use the narrowest useful `cargo check` for the affected crate or product surface.
  - Web UI: use `pnpm run type-check:web` for meaningful TS/React logic changes.
  - Locales: use `pnpm run check:i18n` when locale files or keys change.
  - Design system: use `pnpm run check:design-system` when reusable design-system contracts change.
- If a higher-level check already compiles the touched lower-level crate, do not also run the lower-level check.
- Run tests only for changed behavior, using exact test names or the narrowest useful filter.
- Use focused E2E only for high-risk product flows, cross-surface integration, repeated regressions, or when requested.
- Avoid full builds, broad test suites, desktop builds, web builds, and full E2E unless requested, release-critical, or cheaper checks are insufficient.

## Critical Rules

### Platform Boundaries

Core code must stay platform agnostic.

- In `src/crates/core`, do not depend on Tauri types such as `tauri::AppHandle`.
- Prefer `sparo_events::EventEmitter`, service traits, and constructor-injected dependencies.
- Desktop-specific code belongs under `src/apps/desktop` or an adapter layer.
- Keep Tauri command DTOs and shared command request/response structs structured and serializable.

### Tauri Commands

Command names are `snake_case` in Rust and invoked as `camelCase` through TypeScript helpers when exposed to the UI.

Always prefer a structured request object:

```rust
#[tauri::command]
pub async fn your_command(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GCWing/Sparo-OS](https://github.com/GCWing/Sparo-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
