---
trigger: always_on
description: This file is the working agreement for agents making changes in ExoRoute. Follow the user's request and these project rules. Complete the pre-change gate below before editing, keep changes focused, preserve existing behavior unless the task calls for changing it, and report checks that could not be run.
---

# ExoRoute Agent Guide

This file is the working agreement for agents making changes in ExoRoute. Follow the user's request and these project rules. Complete the pre-change gate below before editing, keep changes focused, preserve existing behavior unless the task calls for changing it, and report checks that could not be run.

## Project at a glance

ExoRoute is a Rust AI protocol gateway with an Axum HTTP server, a Svelte 5 / TypeScript single-page dashboard, and an embedded LMDB environment. The Rust binary embeds the built dashboard from `web/dist` with `RustEmbed`.

- `src/main.rs`: startup, CLI, server construction, and middleware.
- `src/config.rs`: environment and file configuration.
- `src/state.rs`: shared application state and initialized services.
- `src/db.rs`: LMDB initialization, process lock, and typed domain storage operations.
- `src/storage.rs`: the only module that opens and operates on the LMDB environment.
- `src/admin.rs`: authenticated dashboard API.
- `src/gateway.rs`: public model API, provider selection, retries, streaming, and request logs.
- `src/egress.rs`: outbound provider URL validation and DNS/IP protections.
- `src/security.rs`, `src/rate_limit.rs`, `src/client_ip.rs`: authentication and request-safety controls.
- `src/protocol.rs`: protocol conversion between supported client and provider formats.
- `src/static_assets.rs`: serving the embedded frontend.
- `web/src/`: Svelte application, shared API/types/i18n modules, and components.
- `scripts/`: maintenance and benchmark tools.
- `.github/workflows/release.yml`, `build.bat`: release and Windows build workflows.

The application stores its `.env` file and LMDB environment under the current user's `~/.exoroute` directory (on Windows, `%USERPROFILE%\.exoroute`); the default environment path is `exoroute.lmdb`. `EXOROUTE_DATABASE_PATH`, when set, names the LMDB environment directory. Environment variables override values from the app `.env` file. Relative data paths follow the resolution rules in `src/config.rs`.

The database is local-filesystem-only and is owned by one ExoRoute process at a time. Never put it on a network filesystem or open it from another process. Existing SQLite files are intentionally ignored: ExoRoute does not migrate SQLite databases or accept SQLite backups. Do not delete, inspect, or repurpose an old SQLite file as part of an LMDB change.

Use `README.md`, the current code, and existing tests as the source of truth when details differ from this guide. Do not assume that a planned or documented feature is implemented without checking it.

## Working in the codebase

- Inspect neighboring code and reuse its established patterns before adding abstractions, dependencies, or parallel implementations.
- Keep Rust handlers, database operations, and protocol conversion in their existing modules. Keep frontend HTTP calls and shared types in `web/src/lib/` where appropriate.
- Make API changes consistently across the Rust handler, frontend client/types, and tests. Keep user-facing text translated in both English and Vietnamese in `web/src/lib/i18n.ts`.
- Match the current Svelte and CSS conventions. The project uses Svelte 5 but existing components may use legacy reactive syntax; do not migrate components to runes as part of unrelated work.
- Avoid adding a UI framework or a large dependency for a small feature. If a dependency is necessary, use a maintained, compatible package and explain why it is needed.
- Preserve existing error handling and return useful, non-secret diagnostics. Do not silently discard unsupported user input when the API can report that it is unsupported.
- Do not make production-throughput claims from unit tests or the benchmark script alone. State the test environment and measured limits.

## Mandatory pre-change gate

Before writing code, an agent must establish what it is changing and what behavior must remain true.

1. Read the relevant request and this guide. Inspect `git status` and the existing diff first. Treat modified and untracked files as user work: do not overwrite, stage, revert, or reformat unrelated changes.
2. Trace the affected behavior through its callers, handlers, persistence, UI/API types, configuration, and tests. Read neighboring code and use existing patterns before proposing a new abstraction or dependency.
3. For Rust work, check the active toolchain, `Cargo.toml`, `Cargo.lock`, build scripts, features, CI/release workflow, and migrations when applicable. For UI work, inspect the feature boundary, shared types, translation dictionaries, and existing Svelte/CSS conventions.
4. Before editing, identify the intended behavior, input and output contracts, security/data invariants, ownership/state boundary, failure and cancellation paths, and how existing installations or clients remain compatible. For non-trivial work, communicate a short implementation plan and important risks before editing, then proceed without waiting for approval unless the request itself requires approval.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nghiaomg/ExoRoute](https://github.com/nghiaomg/ExoRoute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
