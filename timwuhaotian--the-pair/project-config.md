---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

`AGENTS.md` contains the longer architectural reference (module table, component table, state machine, release process). Read it when you need that depth; this file covers the essentials for getting productive fast.

## What this app is

The Pair is a Tauri 2 desktop app (Rust backend, React 19 + TypeScript frontend) that orchestrates two AI agents working together on a coding task:

- **Mentor** — read-only planner/reviewer
- **Executor** — code writer / command runner

The Rust backend spawns local provider CLIs (`opencode`, `claude`, `codex`, `gemini`), parses their JSON event streams, runs a turn-based state machine, monitors process resources, and tracks git changes. The React frontend renders the conversation, status, and controls.

## Commands

```bash
# Setup
npm install
npm run preflight              # Verify rust/node/CLI tools are present

# Dev (runs Tauri shell + Vite renderer on :5173)
npm run dev                    # Uses scripts/run-with-rustup.mjs to inject PATH
npm run dev:mock               # Same, with THE_PAIR_E2E_MOCK=true (no real CLI spawns)
npm run dev:renderer           # Renderer-only (browser at :5173, no Tauri APIs)

# Quality gates (run before committing)
npm run lint                   # eslint --cache
npm run typecheck              # Runs both typecheck:node and typecheck:web
npm test                       # JS tests then `cargo test` for src-tauri
npm run test:js                # node --test against tests/*.test.ts (tsx loader)
npm run test:rust              # cargo test in src-tauri

# Run a single JS test file
node --import tsx --test tests/handoffGuard.test.ts

# Run a single Rust test
cargo test -q --manifest-path src-tauri/Cargo.toml <test_name>

# Builds (each runs preflight + ensures rust targets first)
npm run build:mac              # universal-apple-darwin DMG
npm run build:mac:release      # Release ZIP bundle used in GitHub Releases
npm run build:win              # x86_64-pc-windows-msvc
npm run build:linux            # x86_64-unknown-linux-gnu

# E2E (Appium + WebdriverIO, macOS only, mock mode)
npm run e2e:setup              # One-time: appium driver install mac2
npm run e2e

# Release
npm run bump <version>         # Updates package.json + Cargo.toml + tauri.conf.json
npm run validate:changelog
```

`npm test` does **not** invoke the renderer or run e2e — `tests/*.test.ts` are pure-Node unit tests covering lib utilities, store logic, and build scripts. Renderer components are tested by reasoning about pure helpers extracted out of them (see `tests/dashboardPairs.test.ts`, `tests/pairListSection.test.ts`).

## Architecture in 60 seconds

```
src/renderer/src/          React 19 frontend
  components/              UI (PascalCase.tsx); primitives live in components/ui/
  store/                   Zustand stores: usePairStore, useUpdateStore, useThemeStore, useLocaleStore
  lib/                     Pure helpers — testable without React. tauri-api.ts wraps invoke()
  hooks/                   Cross-component React hooks
  locales/  i18n.ts        i18next setup (en/zh/ja/ko)
  assets/main.css          Tailwind v4 + theme tokens (light/dark)

src-tauri/src/             Rust backend (see AGENTS.md for the full module table)
  lib.rs                   Registers every Tauri command in invoke_handler![]
  pair_manager.rs          Pair CRUD + task assignment
  message_broker.rs        Turn-coordination state machine
  process_spawner.rs       Spawns provider CLIs, parses JSON event streams
  provider_adapter.rs      Per-provider Input/Output/Session/Permission/Cwd strategies
  provider_registry.rs     Detect installed CLIs + their model catalogs
  session_snapshot.rs      Persist/restore full pair state
  resource_monitor.rs      Per-agent CPU/memory polling
  git_tracker.rs           Diff vs baseline commit
  recent_activity.rs       Recent activity feed shown on the dashboard

tests/                     Node --test unit tests (TypeScript via tsx)
e2e/                       Appium + WDIO end-to-end specs (macOS, mock mode)
scripts/                   Node-based build/release tooling
```

### How frontend talks to backend

Always go through `src/renderer/src/lib/tauri-api.ts`, which wraps `invoke()` from `@tauri-apps/api/core` with typed helpers. The renderer never imports Node built-ins; the only escape hatch is `tauri-shim.ts`, which provides safe fallbacks when running under `npm run dev:renderer` (no Tauri host).

### Adding a Tauri command

1. Implement `#[tauri::command]` in the appropriate `src-tauri/src/<module>.rs`.
2. Add the function to the `tauri::generate_handler![]` list in `src-tauri/src/lib.rs`.
3. Add a typed wrapper in `src/renderer/src/lib/tauri-api.ts`.
4. Cover with a unit test in `tests/` (pure logic) and/or a Rust test in the module file.

### State machine

`Idle → Mentoring → Executing → Reviewing → (loop | Finished | Paused | Awaiting Human Review | Error)`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timwuhaotian/the-pair](https://github.com/timwuhaotian/the-pair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
