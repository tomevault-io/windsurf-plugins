---
trigger: always_on
description: VibePet is a Swift Package targeting macOS 14 with Swift tools 6.0. Core reusable, UI-independent code lives in `VibePetCore/`, organized by concern: `Bridge/`, `Adapters/`, `Install/`, `Persistence/`, `Geometry/`, and `Pet/`. Executable targets are split into `VibePetApp/`, `VibePetHooks/`, and `VibePetSetup/`. Tests live under `Tests/` (`VibePetCoreTests/`, `VibePetAppTests/`, `VibePetSetupTests/`, `E2E/`); shared core helpers are under `Tests/VibePetCoreTests/Support/`. Long-lived product doc
---

# Repository Guidelines

## Project Structure & Module Organization

VibePet is a Swift Package targeting macOS 14 with Swift tools 6.0. Core reusable, UI-independent code lives in `VibePetCore/`, organized by concern: `Bridge/`, `Adapters/`, `Install/`, `Persistence/`, `Geometry/`, and `Pet/`. Executable targets are split into `VibePetApp/`, `VibePetHooks/`, and `VibePetSetup/`. Tests live under `Tests/` (`VibePetCoreTests/`, `VibePetAppTests/`, `VibePetSetupTests/`, `E2E/`); shared core helpers are under `Tests/VibePetCoreTests/Support/`. Long-lived product docs are in `docs/` (`VibePet-PRD.md`), current-version design in `docs/superpowers/specs/`, archived docs in `docs/archive/`; OpenSpec requirements and archived changes are in `openspec/`.

## where to 

## Build, Test, and Development Commands

- `swift build` builds all library and executable targets.
- `swift test` runs the `VibePetCoreTests` XCTest suite.
- `swift run VibePetApp` launches the app executable.
- `swift run VibePetSetup` runs local setup behavior.
- `swift run VibePetHooks` runs the hook bridge helper.

Use `swift package describe --type json` when you need to confirm target membership or products.

## Coding Style & Naming Conventions

Use idiomatic Swift with 4-space indentation, `UpperCamelCase` for types, and `lowerCamelCase` for properties, functions, and enum cases. Keep source files focused around one primary type or feature area. Public model types should remain explicit about protocol conformances such as `Codable`, `Equatable`, and `Sendable` when they cross package or bridge boundaries. No repository SwiftLint or SwiftFormat configuration is currently present, so rely on SwiftPM compilation and local consistency.

## Testing Guidelines

Tests use XCTest and should be added under the matching `Tests/` target (`VibePetCoreTests/`, `VibePetAppTests/`, `VibePetSetupTests/`, or `E2E/`) with filenames ending in `Tests.swift`. Follow the existing `test...` method naming pattern, for example `testApprovalContentRoundTrips`. Prefer deterministic fixtures (e.g. `Tests/Fixtures/claude/`) over ad hoc local files. Run `swift test` before submitting changes that affect core logic, bridge serialization, adapters, the installer, persistence, or fail-open paths. Verify installer/config-writer logic by unit tests only — never real install smoke tests, since writes hit the real `~/.codex` / `~/.claude` even with `$HOME` overridden. An intermittent SIGPIPE (signal 13) during a full `swift test` run is not a regression; re-run or use `--filter`.

## Commit & Pull Request Guidelines

Recent history uses short, imperative summaries, sometimes with conventional prefixes such as `feat:`. Keep the first line focused on intent. Include context in the body when behavior, architecture, or requirements change, and use project decision trailers where useful, especially `Constraint:`, `Rejected:`, `Tested:`, and `Not-tested:`. Pull requests should summarize the change, link related OpenSpec items or issues, list verification performed, and include screenshots or recordings for visible app changes.

## Security & Configuration Tips

Do not commit generated build output, private local paths, credentials, or personal fixture data. Keep `.build/` and local tool caches out of reviews. When changing bridge or hook behavior, document any new socket, file-system, or command-execution assumptions in code and tests.

## Reset / First Launch State Cleanup

To make VibePet behave like a fresh first launch, clear the app's persisted state and managed integration footprint:

- Quit VibePet first so `bridge.sock` and in-memory session state are gone.
- Run `swift run VibePetSetup uninstall all` when possible before deleting state. This removes VibePet-managed Claude Code and Codex hook entries while preserving user hooks and config.
- Delete the entire `~/Library/Application Support/VibePet/` directory. This resets onboarding (`hasCompletedOnboarding`), selected pet, pet position, decision timeout, imported pets, session socket, install manifest/backups, and the stable `bin/VibePetHooks` copy.
- On next launch, VibePet should recreate `~/Library/Application Support/VibePet/` from defaults and show onboarding as if no prior app state existed.
- If uninstall cannot run before the directory is deleted, manually remove only hook entries that reference `~/Library/Application Support/VibePet/bin/VibePetHooks` from `~/.claude/settings.json`, and only Codex hook groups marked `statusMessage: "Managed by VibePet"` or referencing that same binary from `~/.codex/hooks.json`.
- In `~/.codex/config.toml`, remove VibePet-managed `[features]` `hooks = true` / `codex_hooks = true` only when no other Codex hooks remain. Do not delete unrelated Codex settings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caichuanwang/VibePet](https://github.com/caichuanwang/VibePet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
