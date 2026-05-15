---
trigger: always_on
description: Zenii is a Rust workspace producing 5 binaries (Desktop, Mobile, CLI, TUI, Daemon) from a
---

# CLAUDE.md -- Zenii

## Project Overview

Zenii is a Rust workspace producing 5 binaries (Desktop, Mobile, CLI, TUI, Daemon) from a
single shared core. All clients communicate via HTTP+WebSocket gateway (axum at 127.0.0.1:18981).

## v2 Philosophy

1. **Use proven crates, don't hand-roll** -- `sysinfo`, `websearch`, `rig-core`, `ignore` over custom implementations.
2. **Port patterns, not code** -- port v1 design (trait-based tools, security policy, memory abstraction) adapted to v2 conventions.
3. **Lean by default** -- feature-gate optional modules. Default binary includes only core operation.
4. **Single shared core** -- ALL business logic in `zenii-core`. Binary crates are thin shells (<100 lines).

## Tech Stack

Rust 2024 | Tokio | rig-core | rusqlite + sqlite-vec | axum | Svelte 5 + Tauri 2 | openclaw-channels | comrak + Tera

## Commands

```bash
cargo check --workspace                      # Compile check
cargo test --workspace                       # Run all tests
cargo clippy --workspace                     # Lint
cargo run -p zenii-daemon                    # Start daemon
cargo run -p zenii-cli -- chat               # CLI chat
cd web && bun run dev                        # Frontend dev
cd crates/zenii-desktop && cargo tauri dev   # Desktop app
./scripts/build.sh --target native --release # Build binaries
```

## Workspace Structure

→ Full module detail in `docs/architecture.md`

```
crates/zenii-core/     # Shared library — ALL business logic, NO Tauri dep
crates/zenii-desktop/  # Tauri 2 shell (macOS, Windows, Linux)
crates/zenii-mobile/   # Tauri 2 shell (iOS, Android)
crates/zenii-cli/      # clap CLI (thin wrapper)
crates/zenii-tui/      # ratatui TUI (thin wrapper)
crates/zenii-daemon/   # Headless daemon (thin wrapper)
web/                   # Svelte 5 frontend (shared by desktop + mobile)
docs/                  # Architecture diagrams, phase details, process flows
plans/                 # Detailed per-phase implementation plans
tests/                 # Per-phase test plans and results
```

## Strict Rules

1. **No std::sync::Mutex in async paths** -- use tokio::sync::Mutex or DashMap
2. **No block_on()** -- use tokio::spawn or .await
3. **No Result<T, String>** -- use ZeniiError enum (thiserror)
4. **All SQLite ops via spawn_blocking** -- rusqlite is sync
5. **Zero business logic in binary crates** -- everything in zenii-core
6. **No code duplication** -- if used twice, extract to zenii-core
7. **TDD gate order**: plan → user approves → write tests → user approves → implement → cargo test → user validates
8. **No phase proceeds without user confirmation at all 3 gates**
9. **All public functions must have unit tests**
10. **Feature flags for optional modules** -- keep default binary lean
11. **Research before adding dependencies** -- compare alternatives, document rationale in `plans/`
12. **Binary size matters** -- prefer lightweight crates, check dependency trees
13. **Never skip the workflow** -- plan file in `plans/` + test plan in `tests/` MUST exist on disk before any `.rs` file is created or modified. No exceptions.

## Conventions

→ Verbose details for credential naming, dark-mode select, and no-magic-numbers in `docs/conventions.md`

- Error handling: `ZeniiError` enum with thiserror, no `.map_err(|e| e.to_string())`
- Async: tokio::sync primitives only, never std::sync in async code
- Concurrency: DashMap for concurrent HashMaps, tokio::sync::Mutex for async locks
- Testing: `#[cfg(test)]` in same file, integration tests in `tests/`; test success + failure paths
- Naming: snake_case (Rust), camelCase (TypeScript/Svelte)
- Imports: std → external crates → internal modules (blank line separated)
- Logging: `tracing` macros only (info!, warn!, error!, debug!), never println!
- Paths: absolute in code, relative when referencing to user
- SQL: parameterized queries only, WAL mode, migrations in transactions
- Security: never log credentials, use zeroize for sensitive data, keyring for storage
- Credential keys: colon-separated namespacing — `api_key:{provider_id}`, `channel:{id}:{field}`
- Structs: derive `Debug, Clone, Serialize, Deserialize` on all public structs
- Enums: `#[non_exhaustive]` on public enums that may grow
- Async locks: never hold across `.await` points
- No magic numbers: all tunables defined as `AppConfig` fields in `schema.rs`
- Native `<select>` dark mode: use `bg-background text-foreground`, never `bg-transparent`

## Agent Usage

- **Explore agents**: broad codebase research, deep traversal, or unfamiliar modules (>3 queries).
- **Parallel task agents**: independent workstreams (e.g., updating unrelated modules simultaneously).
- **Research agents**: dependency research, v1 analysis, documentation lookups.
- **Skip agents**: single Glob/Grep suffices, sequential steps, trivial 1-2 file edits.

Do not duplicate work an agent is already doing. Delegate, then use the results.

## Plan Mode Requirement

Always start in Plan Mode for: new features, new phases, architectural changes, or tasks spanning
multiple files. Enter `EnterPlanMode` first, outline steps, exit only when the plan is clear.

## Phase Gate Workflow

Every phase has 3 user gates — no skipping. → Full checklist in `docs/phases.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sprklai/zenii](https://github.com/sprklai/zenii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
