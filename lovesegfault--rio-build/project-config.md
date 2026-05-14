---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

rio-build is an early-stage Rust project. It uses a Nix flake-based development environment with crate2nix for building Rust packages and protobuf for gRPC code generation.

## Quick Start

The dev environment is managed by Nix. If you have direnv, it activates automatically via `.envrc`.

```bash
# Enter the dev shell (if direnv isn't set up).
# Default shell uses NIGHTLY Rust so cargo-fuzz works; use .#stable for CI parity.
nix develop

# Build
cargo build

# Run
cargo run

# Test (prefer nextest for better output)
cargo nextest run
# or: cargo test

# Single test
cargo nextest run test_name
# or: cargo test test_name

# Lint
cargo clippy --all-targets -- --deny warnings

# Format (uses treefmt: rustfmt + nixfmt + taplo)
treefmt

# Full CI-equivalent checks (clippy, tests, docs, coverage)
nix flake check

# Fuzz a parser (default shell is nightly, so this works directly)
cd rio-nix/fuzz && cargo fuzz run wire_primitives
```

## Build System

- **Nix + crate2nix**: The flake.nix defines the full build pipeline. crate2nix generates per-crate derivations from `Cargo.lock` (JSON output mode — no `Cargo.nix` checked in), giving per-crate caching via nixpkgs' `buildRustCrate`.
- **Cargo workspace**: Root Cargo.toml is a workspace; crates live in subdirectories.
- **Protobuf**: `.proto` files are picked up by the crate2nix source filter. `PROTOC` and `LIBCLANG_PATH` are set automatically in the dev shell.

## Key Commands via Nix

| Command | What it does |
|---|---|
| `nix build` | Build the workspace (release profile with thin LTO) |
| `/nixbuild .#ci` | Full validation: build, clippy, nextest, doc, coverage, pre-commit, 2min fuzz ×9, all VM tests, cov-smoke (Linux+KVM only) |
| `nix flake check` | Runs all `checks.*` (build, clippy, nextest, doc, coverage, 2min fuzz, VM tests) |
| `nix develop .#stable` | Dev shell with stable Rust (CI parity) |
| `nix build .#checks.x86_64-linux.tracey-validate` | Spec-coverage validation (r[...] annotation integrity) |
| `tracey query status` | Spec-coverage summary (in dev shell) |
| `nix fmt` | Same as `treefmt` |
| `/nixbuild .#coverage-full` | Combined unit+VM coverage (lcov+HTML, ~25min, needs KVM) |
| `nix build .#cov-smoke` | Fast (~5min) one-scenario coverage-infra smoke (also in `.#ci`) |
| `/nixbuild .#cov-vm-protocol-warm-standalone` | Run one VM test in coverage mode (debugging, raw profraws at `result/coverage/`) |
| `nix build .#coverage-vm-protocol-warm-standalone` | Per-test lcov from one coverage-mode VM run |

### CI aggregate target

`/nixbuild .#ci` bundles all checks + VM tests + 2min fuzz into a single build. Needs KVM for the VM tests. On non-Linux, degrades to cargo checks + pre-commit only (VM tests and fuzz are both Linux-only). Result is a directory of symlinks to each constituent's output (`ls result/`).

### Coverage

Three tiers:

- **Unit-test only** (~5min): `nix build .#checks.x86_64-linux.coverage`. Output: `result/lcov.info`. HTML via `nix build .#coverage-html`.
- **Cov-smoke** (~5min, in `.#ci`): `nix build .#cov-smoke`. One representative VM scenario in coverage mode, asserts profraw→lcov pipeline produced non-empty data. Catches "coverage infrastructure broken" at merge-gate. A PSA break went 118 commits undetected before this was added — coverage-full failures were triaged as individual test-gaps instead of a pipeline-level halt.
- **Combined unit+VM** (~25min, needs KVM): `/nixbuild .#coverage-full`. Output: `result/lcov.info` (combined), `result/html/`, `result/per-test/vm-<scenario>-<fixture>.lcov`. Fills the ~15% "permanently red" gap of VM-only code (FUSE callbacks, namespace setup, cgroup tracking, main.rs wiring, k8s lease/reconcilers, SSH accept loop). **Not** in `.#ci` — run on demand.

VM coverage architecture details: see `.claude/rules/coverage.md` (loads when editing `nix/coverage.nix`).

## Formatting

treefmt runs three formatters:
- **rustfmt** for `.rs` files
- **nixfmt** for `.nix` files
- **taplo** for `.toml` files

Pre-commit hooks run treefmt automatically on commit.

## Development Notes

- Rust edition is **2024** — use latest Rust idioms and features.
- Clippy is configured with `--deny warnings` — all warnings must be fixed.
- The `target/` directory is gitignored; Nix builds go to `result`/`result-*` (also gitignored).
- Integration tests may need `nix` available in PATH (it's provided in the dev shell).
- **Default dev shell uses nightly Rust** so `cargo fuzz` works directly. CI builds (clippy, nextest, workspace) use stable via `rust-toolchain.toml` — nightly-only code will be rejected by `nix flake check`. Use `nix develop .#stable` for CI-parity development.
- **Always run cargo commands via `nix develop -c`** to ensure all dev shell dependencies (including fuse3) are available. E.g., `nix develop -c cargo nextest run`, `nix develop -c cargo clippy --all-targets -- --deny warnings`.
- **Always run `nix develop -c cargo nextest run` before committing** to catch regressions early.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lovesegfault/rio-build](https://github.com/lovesegfault/rio-build) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
