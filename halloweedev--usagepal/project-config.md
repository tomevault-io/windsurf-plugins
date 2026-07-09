---
trigger: always_on
description: Version: 0.31 (2026-06-10)
---

# AGENTS.md

Version: 0.31 (2026-06-10)

> OpenUsage is a public-facing Tauri desktop app for tracking AI provider usage across plugins.

## Roadmap: Windows & Linux Support

This is a Tauri/Rust fork of OpenUsage. The upstream Swift rewrite is **not** happening here — development stays on Tauri/Rust, and the goal is to ship **Windows and Linux alongside macOS, both as first-class targets**.

Current state: macOS-only. The build matrix (`.github/workflows/publish.yml`) and bundle targets (`tauri.conf.json`) produce Mac builds only, and the app relies on macOS-native APIs that don't yet compile elsewhere.

Major work to get cross-platform (roughly largest-first):
- **Tray dropdown panel** — the core click-tray-to-open-panel UX is built on macOS `NSPanel` (`src-tauri/src/panel.rs`, `tauri-nspanel`, objc2, `macos-private-api`). Rebuild it as a borderless, always-on-top, non-activating window with per-OS tray positioning and hide-on-blur. Biggest single item.
- **Linux tray** — libayatana-appindicator often can't report the icon's screen position and some desktops only support a right-click menu; making the panel anchor well across desktop environments is the main Linux risk.
- **Plugin credential/usage paths** — ~20 plugins hardcode macOS paths (`~/Library/Application Support/…`, `~/.config`, `~/.local/share`). Each provider needs its Windows (`%APPDATA%`/`%LOCALAPPDATA%`) and Linux equivalents added and verified. Keychain reads throw off-macOS; add Windows Credential Manager / Linux Secret Service for parity (file-fallback plugins degrade gracefully without it).
- **Packaging & CI** — add `windows-latest` + `ubuntu-latest` to the build matrix (Linux needs webkit2gtk + appindicator system libs), and add `nsis`/`msi` and `deb`/`appimage` bundle targets. The updater, `latest.json`, and signing key are already cross-platform. Windows needs a code-signing certificate (~$200–400/yr, or Azure Trusted Signing) to avoid SmartScreen warnings; Linux needs none.
- **Small platform shims** — `open_notification_settings` and the dock/activation-policy code are macOS-only and need Windows/Linux equivalents or graceful no-ops. Notifications already have a non-macOS branch.

### Release hygiene
- Never leave a release in Draft. After every release, verify it is published with its assets (use the release-tauri skill).

## Documentation

- Logic changes must update any docs in `docs/` that describe the affected behavior.
- Plans must list the doc files that need updating as part of the work.
- Exclude design from docs, and keep them simple, less-technical, easy to skim.

## Guardrails

- Use `trash` for deletes
- Use `mv` / `cp` to move and copy files
- Bugs: add regression test when it fits
- Keep files <~500 LOC; split/refactor as needed
- Before writing code, strictly follow the below research rules

## IPC Types (specta)

Tauri IPC types are auto-generated from Rust to TypeScript via `tauri-specta`. Rust is the single source of truth — never hand-write TS types that mirror Rust IPC structs.

- **Generated file:** `src/bindings.ts` (do not edit by hand). Contains all command signatures, event types, and type definitions.
- **When you change a Rust IPC type or command:** add `#[derive(specta::Type)]` to new structs/enums, `#[specta::specta]` to new `#[tauri::command]` functions, and `#[derive(tauri_specta::Event)]` + `#[tauri_specta(event_name = "...")]` to new event types. Then register them in the `Builder::new().commands(...).events(...)` call in `src-tauri/src/lib.rs` `run()` (and in the `export_bindings` test in the same file).
- **Regenerate bindings:** `cd src-tauri && cargo test test_export_bindings`. The `run()` function also auto-exports on every debug-mode app launch (`npm run tauri dev`), so bindings stay in sync during development.
- **Frontend usage:** import types from `@/bindings` (e.g., `import type { MetricLine, PluginOutput } from "@/bindings"`). Re-export them through `src/lib/plugin-types.ts` if you need narrowed literal types. Keep using `invoke<T>("command_name", args)` and `listen<T>("event_name", handler)` — do not switch to the generated `commands.foo()` wrappers (they use a `typedError` pattern that would break existing try/catch blocks and test mocks).
- **specta forbids `u64`/`usize`/`i64`/`isize`** in IPC types (BigInt precision loss in JS). Use `f64` for timestamps, byte sizes, and counts that cross the IPC boundary — safe within JS's 2^53 safe integer range.
- **`Option<T>` becomes `T | null`** in generated TS (not `T | undefined`). Frontend code must handle `null`, not just `undefined`. Use `?? defaultValue` or null guards.

## Research

- Check for and prefer available skills over web research.
- Prefer researched knowledge over your own knowledge when skills are unavailable.
- Research: Exa to general search, Context7 for official docs, GitHits for open source examples
- Best results: Quote exact errors; prefer late-2025/2026+ sources.

## Error Handling

Always fail loudly into error logging (e.g., Sentry) and but show friendly errors to the user. Do not add silent fallbacks that hide real problems.

## UI

Always use titlecase any hardcoded copy for titles.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Halloweedev/usagepal](https://github.com/Halloweedev/usagepal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
