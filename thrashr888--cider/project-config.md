---
trigger: always_on
description: `src/lib.rs` is the library root and `src/main.rs` is the Clap CLI over it — the binary is a front-end, and every command's real work lives in a `sources` function. Keep it that way: logic that lands in `main.rs` is logic a library consumer (Alchemy links this crate) cannot reach. Clap belongs to the `cli` feature, on by default for the binary and off for library consumers, so nothing under `src/sources/` may depend on it. `src/sources/*.rs` holds one module per Apple app or data source, such as
---

# Repository Guidelines

## Project Structure & Module Organization
`src/lib.rs` is the library root and `src/main.rs` is the Clap CLI over it — the binary is a front-end, and every command's real work lives in a `sources` function. Keep it that way: logic that lands in `main.rs` is logic a library consumer (Alchemy links this crate) cannot reach. Clap belongs to the `cli` feature, on by default for the binary and off for library consumers, so nothing under `src/sources/` may depend on it. `src/sources/*.rs` holds one module per Apple app or data source, such as `reminders.rs` or `calendar.rs`; keep new integrations in snake_case and export them from `src/sources/mod.rs`. `src/sources/util.rs` contains shared subprocess, timeout, and date helpers. `src/pretty.rs` owns human-readable `--pretty` rendering. Repo-specific agent skills live in `.agents/skills/`, and the end-user CLI skill lives in `.skills/`. `.github/workflows/` contains CI and release automation. `target/` is generated build output and should stay untracked.

## Build, Test, and Development Commands
Use `cargo run -- reminders --pretty` to exercise a single command locally. Run `cargo build --release` to produce the binary at `target/release/cider`. Run `cargo test` for the repository’s inline unit tests (they live on the lib target). Also build the way a library consumer does — `cargo clippy --lib --no-default-features -- -D warnings` — which is what catches a Clap dependency creeping into `src/sources/`. Before opening a PR, match CI with `cargo fmt -- --check` and `cargo clippy -- -D warnings`. CI runs on `macos-latest`, so macOS-specific behavior should be verified there.

## Coding Style & Naming Conventions
Follow `rustfmt` defaults: 4-space indentation, standard import grouping, and formatter-controlled line wrapping. Use `snake_case` for files, modules, functions, and JSON fields; use `UpperCamelCase` for structs and enums. Prefer small source-specific modules and reuse helpers from `src/sources/util.rs` instead of duplicating AppleScript, JXA, or timeout logic. Keep stdout machine-readable JSON and send diagnostics to stderr; any `--pretty` presentation logic belongs in `src/pretty.rs`.

## Testing Guidelines
This repo keeps unit tests inline with implementation under `#[cfg(test)] mod tests`. The one file under `tests/` is the live smoke suite, `tests/live.rs`: it runs every read-only command against the Mac it is on and fails if a command hangs, prints something other than JSON, exits non-zero without a typed error envelope, or returns an empty list for a store that `cider doctor` reports as present (the bug that let `find-my` ship returning `[]`). It is `#[ignore]`d because CI runners have no user data and a JXA call can open a permission dialog; run it yourself before a release with `cargo test --test live -- --ignored --nocapture`. It is read-only by construction: a guard refuses any subcommand outside a short allowlist of read verbs, so it can never create, change, or delete anything on the machine. Keep it that way when adding cases. A second section covers the Cider Bridge read paths (`bridge status`, `home --live homes|scenes`, `home state`, `home triggers`, `weather`, `permissions`, `icloud quota`) only when the app is installed; its one side effect is launching the bridge app for `--live`, which it announces with a `live: launching Cider Bridge` line and never undoes (it leaves the app running). Add deterministic unit tests for parsing, formatting, schema changes, and action result shapes. Avoid tests that require live Apple app state unless there is no stable alternative. When changing output, cover both the JSON contract and any affected pretty rendering.

## Commit & Pull Request Guidelines
Recent history uses short, imperative commit subjects, often with Conventional Commit prefixes like `feat:` and `fix:`; release commits use `Bump to vX.Y.Z`. Keep commits focused and easy to scan. PRs should explain which commands or source modules changed, list validation performed (`cargo test`, `cargo clippy`, manual command runs), and include sample command/output snippets for user-facing CLI changes. Call out any macOS permission or side-effect implications for mutating commands.

## Release Process
Releases are automated via `.github/workflows/release.yaml`, triggered when a `v*` tag appears. The workflow builds macOS binaries (aarch64 + x86_64), creates a GitHub release with tarballs, publishes to crates.io, and updates the Homebrew tap.

To cut a release:
1. Bump `version` in `Cargo.toml` and commit: `Bump to vX.Y.Z`
2. Push the commit to `main`
3. Create the release as a **draft**, with the notes it should ship with:
   ```sh
   gh release create vX.Y.Z --target main --draft --notes "..."
   ```
4. Create the tag, which is what actually triggers the workflow:
   ```sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thrashr888/cider](https://github.com/thrashr888/cider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
