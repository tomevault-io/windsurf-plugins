---
trigger: always_on
description: Surge is a Cargo workspace plus a .NET wrapper:
---

# Repository Guidelines

## Project Structure & Module Organization
Surge is a Cargo workspace plus a .NET wrapper:
- `crates/surge-core/`: update engine (storage, releases, diff, pack, update, supervisor).
- `crates/surge-cli/`: `surge` CLI.
- `crates/surge-ffi/`: C ABI used by native/.NET callers.
- `crates/surge-installer/`: console installer launcher (extracts payload, delegates to `surge setup`).
- `crates/surge-installer-ui/`: GUI installer with egui (self-contained graphical installer).
- `crates/surge-supervisor/`: supervisor binary.
- `dotnet/`: managed wrapper and tests (`Surge.NET`, `Surge.NET.Tests`).
- `include/surge/`: public C headers.
- `vendor/bsdiff/`: required submodule for C bsdiff backend.
- `crates/surge-core/vendor/`: committed publishable snapshot generated from `vendor/bsdiff` for `surge-core` builds and crates.io packaging.
- `assets/`, `demoapp/`: examples and fixtures.

### Installer Types
Surge supports four installer types configured via `installers:` in the manifest:
- `online`: Console installer that downloads the package at install time (uses `surge-installer`).
- `offline`: Console installer with the full package embedded (uses `surge-installer`).
- `online-gui`: GUI installer (egui) that downloads at install time (uses `surge-installer-ui`).
- `offline-gui`: GUI installer (egui) with the full package embedded (uses `surge-installer-ui`).

The legacy `web` type has been removed; use `online` instead.

## Build, Test, and Development Commands
Initialize submodules first:
```bash
git submodule update --init --recursive
```
Rust:
```bash
RUSTFLAGS="-D warnings" cargo build --release
RUSTFLAGS="-D warnings" cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
```
.NET:
```bash
dotnet build dotnet/Surge.slnx --configuration Release
dotnet test dotnet/Surge.slnx --configuration Release
```

- Run the workspace validation commands non-interactively. Do not allocate a TTY/PTY for `cargo test --workspace` or the chained pre-push validation suite, because prompt-path tests can wait on an interactive terminal and hang the run.

## Mandatory Pre-Push Validation
Before any push, run the same quality gates CI uses. Do not push if any command fails.

```bash
./scripts/sync-surge-core-vendor.sh --check
./scripts/check-version-sync.sh
cargo fmt --all -- --check
RUSTFLAGS="-D warnings" cargo test --workspace
cargo clippy --all-targets --all-features -- -D warnings
cargo clippy --workspace --lib --bins --examples -- -D warnings -D clippy::unwrap_used -D clippy::expect_used
cargo clippy --workspace --all-targets --all-features -- -D warnings -W clippy::pedantic
dotnet format dotnet/Surge.slnx --verify-no-changes
dotnet test dotnet/Surge.slnx --configuration Release
```

If the local environment cannot run a listed command, document the exact gap in the PR and run it in CI before merge.

- When updating `vendor/bsdiff` or anything under `crates/surge-core/vendor/`, regenerate the publishable snapshot with `./scripts/sync-surge-core-vendor.sh` before running the checks above.

## Rust Quality Bar (Best Practices)
- Prefer self-documenting code: clear types, names, and small functions over explanatory comments.
- Use comments sparingly; add them only for invariants, non-obvious tradeoffs, or safety contracts.
- Keep modules cohesive and APIs explicit (`Result<T, E>`, typed structs/enums instead of ad-hoc tuples).
- Treat roughly 600 production lines as the point to split a Rust source file; keep module roots orchestration-focused and move detailed behavior into focused leaf modules.
- Prefer typed error enums (`thiserror`) over `Box<dyn Error>` in binaries/crates where error cases are known.
- Consolidate repeated crate-local helpers (for example mutex poison recovery and C-string sanitization) into a shared internal module.
- Prefer `unwrap_or_else(std::sync::PoisonError::into_inner)` over manual `match` when recovering poisoned mutexes.
- Avoid unnecessary `crate::` path prefixes in module-local code/tests when imports already provide the item.
- For multi-app manifests, always scope storage access and emitted installer metadata prefixes by app id; never mix base-prefix release indexes with app-scoped artifact flows.
- For CLI commands that accept both optional `--app-id` and `--rid`, use a shared RID-hint resolver to infer app id only when the RID uniquely identifies one app.
- Minimize `unsafe`: isolate it to FFI/boundary layers, prefer safe wrappers, and remove unnecessary `unsafe impl`.
- Every remaining unsafe block must include a short `SAFETY:` rationale.
- Run periodic panic-path sweeps in non-test targets with:
  - `cargo clippy --workspace -- -D warnings -D clippy::unwrap_used -D clippy::expect_used`
  - fix runtime `unwrap/expect` in production/build paths instead of suppressing lints.
  - `expect_used` is treated the same as `unwrap_used`: both can hide panic paths in runtime code.
- CI hardening tiers:
  - blocking: `cargo clippy --workspace --lib --bins --examples -- -D warnings -D clippy::unwrap_used -D clippy::expect_used`
  - advisory debt visibility: `cargo clippy --workspace --all-targets --all-features -- -D warnings -W clippy::pedantic`
  - keep pedantic advisory until backlog is reduced; then promote selected pedantic lints to blocking.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fintermobilityas/surge](https://github.com/fintermobilityas/surge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
