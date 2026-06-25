---
trigger: always_on
description: - Rust workspace lives under `crates/`: `mergegate-cli` (CLI entry), `mergegate-tui` (UI), and `mergegate-core` (shared logic, config, sessions, MergeGate/DTP: `gate`, `store`, `lock`, `protocol`). Shared workspace config in `Cargo.toml`.
---

# Repository Guidelines

## Project Structure & Module Organization
- Rust workspace lives under `crates/`: `mergegate-cli` (CLI entry), `mergegate-tui` (UI), and `mergegate-core` (shared logic, config, sessions, MergeGate/DTP: `gate`, `store`, `lock`, `protocol`). Shared workspace config in `Cargo.toml`.
- ルートに `package.json` はない（過去テンプレート由来の記述は削除済み）。フロント系ツールが別ディレクトリにあれば、その README に従う。
- Supporting assets: `.claude/` agent configs, `docs/` for design notes, `project_memory/` for Polaris タスク台帳（`tasks.json`）、`.github/` for CI workflows。

## Build, Test, and Development Commands
- Rust: `cargo build --workspace --release` (release build), `cargo test --all` (unit/integration), `cargo clippy --all-targets -- -D warnings` (lint), `cargo fmt --all` (format).
- Toolchain: `rust-version` は `Cargo.toml` の workspace メタデータに準拠（現状 1.75+）。

## Coding Style & Naming Conventions
- Rust: run `cargo fmt` before commits; clippy must be clean with `-D warnings`. Modules/files use `snake_case`, types/traits `PascalCase`, constants `SCREAMING_SNAKE_CASE`. Prefer `anyhow::Result` and `thiserror` for errors; avoid `unwrap` in non-test code.

## Testing Guidelines
- Rust tests colocated in each crate (`mod tests` blocks or `tests/` directories); write integration tests when touching cross-crate behavior. Run `cargo test --all` before PRs.

## Commit & Pull Request Guidelines
- Use Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`). Keep subject under ~72 chars; include scope when helpful (`feat(tui): add diff renderer`).
- PRs should describe the change, linked issue, and test evidence (`cargo test --all`). Add screenshots or terminal recordings for TUI changes when relevant. Keep diffs focused; separate refactors from feature work.

## Environment & Configuration
- Environment defaults from `.env.example`; typical variables: `GITHUB_TOKEN`, `ANTHROPIC_API_KEY`, `REPOSITORY`, optional `RUST_LOG`/`RUST_BACKTRACE` for debugging.
- Local config/state lives in `~/.miyabi/` and `.miyabi/` (do not commit secrets). Review `.miyabi.yml` for runtime behavior before modifying agent workflows.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **miyabi-cli-standalone** (5866 symbols, 12893 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/miyabi-cli-standalone/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring

- **Renaming**: MUST use `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` first. Review the preview — graph edits are safe, text_search edits need manual review. Then run with `dry_run: false`.
- **Extracting/Splitting**: MUST run `gitnexus_context({name: "target"})` to see all incoming/outgoing refs, then `gitnexus_impact({target: "target", direction: "upstream"})` to find all external callers before moving code.
- After any refactor: run `gitnexus_detect_changes({scope: "all"})` to verify only expected files changed.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Tools Quick Reference

| Tool | When to use | Command |
|------|-------------|---------|
| `query` | Find code by concept | `gitnexus_query({query: "auth validation"})` |
| `context` | 360-degree view of one symbol | `gitnexus_context({name: "validateUser"})` |
| `impact` | Blast radius before editing | `gitnexus_impact({target: "X", direction: "upstream"})` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShunsukeHayashi/mergegate](https://github.com/ShunsukeHayashi/mergegate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
