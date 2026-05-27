---
trigger: always_on
description: Workspace crates live in `frontend/`, `backend/`, `shared/`, and `src-tauri/`, compiled via the root `Cargo.toml`. Actor+Relay specs sit in `docs/actors_relays/actor_relay_architecture.md` with migration notes in `docs/actors_relays/novywave/migration_strategy.md`. Domain responsibilities are charted in `.claude/extra/project/domain_map.md`, while full product specs live in `.claude/extra/project/specs/specs.md`. Static fonts ship from `public/`, and waveform fixtures for manual checks are in `t
---

# Repository Guidelines

## Project Structure & Key References
Workspace crates live in `frontend/`, `backend/`, `shared/`, and `src-tauri/`, compiled via the root `Cargo.toml`. Actor+Relay specs sit in `docs/actors_relays/actor_relay_architecture.md` with migration notes in `docs/actors_relays/novywave/migration_strategy.md`. Domain responsibilities are charted in `.claude/extra/project/domain_map.md`, while full product specs live in `.claude/extra/project/specs/specs.md`. Static fonts ship from `public/`, and waveform fixtures for manual checks are in `test_files/`.

## Actor+Relay Architecture
No raw mutables—domain state must flow through Actors, Relays, or Atoms. Name relays for observed events (`file_loaded_relay`, not `load_file`). Cache current values only inside actor loops, react to relay input, and model concrete domains (`TrackedFiles`, `WaveformTimeline`). Prefer ConnectionMessageActor patterns for cross-component messaging (`.claude/extra/architecture/connection-message-actor-pattern.md`). Never introduce fallback values; expose explicit loading or empty states instead. Revisit `.claude/extra/architecture/actor-relay-patterns.md` and `frontend/src/dataflow/` before touching dataflow code.

## Build, Test, and Development Commands
Run `makers install` once to install MoonZoon, the WASM target, and the Tauri CLI. Use `makers start` for the hot-reload web server and read rebuild status from its live terminal output. `makers tauri` reuses that server for desktop debugging; `makers tauri-build` emits bundles. Ship web builds with `makers build`; use `makers clean` before regenerating artifacts.

## Compilation & UI Verification Workflow
Rely on the active `makers start`, `makers watch_plugins`, and `makers tauri` terminals for build status. Wait for the live output to show the relevant successful rebuild, and report every warning or `error[E...]` line explicitly. If another maintainer owns the process, have them share the relevant terminal output directly instead of redirecting to repo log files. For UI verification, use live Tauri bridge inspection, DOM/canvas captures, and console checks rather than OS-level screenshots. If compilation stalls or fails, stop and surface the minimal live output excerpt showing the failure rather than attempting unrelated rebuilds.
When debugging, keep console output lean: add temporary `println!/log!` sparingly and remove them before finishing.

Do not run `cargo check`, `cargo build`, or other cargo compilation commands locally unless this document explicitly instructs you to. Use the supported `makers` workflows and the live dev-process output for build status instead of ad-hoc cargo builds.

Always review the latest live rebuild output after making changes. If it shows any compilation errors or warnings, either fix them immediately or add a TODO documenting the follow-up work before proceeding.

## Coding Style & Testing
Use rustfmt defaults (`cargo fmt --all`), keep modules snake_case and exported types in PascalCase, and prefer derived traits (`#[derive(Default, Clone, Copy, Debug)]`) over manual impls. Avoid suppressing lints (`#[allow]`) except for documented dataflow APIs and preserve existing business logic during refactors. Express absence with `Option` instead of sentinel values. Run `cargo clippy --workspace --all-targets` plus `cargo test --workspace` before PRs; refresh fixtures in `test_files/` when parser logic changes. For UI or Tauri updates, smoke-test with the running `makers tauri` session and its live console output.

## Commit & Pull Request Guidelines
Commits are brief sentence-case summaries (`Implement dock-responsive panel layout`). Explain non-obvious choices in the body and link specs or issues. Pull requests should cover user impact, note completed checks (`cargo fmt`, `cargo clippy`, `cargo test --workspace`), and attach direct UI evidence when relevant.

## Agent Notes
- When the user asks to "create todos", log them in the Codex CLI todo tool instead of modifying repository files (e.g., avoid writing TODOs into `.novywave`).
- Never fully revert or reset ongoing work unless the user explicitly requests and confirms it; prefer incremental corrections that preserve recent changes.
- Avoid blocking config saves on extra "timeline ready" flags or similar coordination in `frontend/src/config.rs`; that pattern has repeatedly left the backend stuck logging `timeline query start` without ever sending `UnifiedSignalResponse`. Restore timeline state by replaying the saved values through existing relays without delaying the config saver.
- When restoring timeline state, guard the bounds listener so a transient `None` range doesn’t wipe the restored viewport—set a flag once config playback runs and ignore the empty bounds ping unless no variables remain.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NovyWave/NovyWave](https://github.com/NovyWave/NovyWave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
