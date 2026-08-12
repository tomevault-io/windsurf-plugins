---
trigger: always_on
description: **Git commit rules:**
---

# CLAUDE.md - Scalable Web3 Storage

## Agent Rules

**Git commit rules:**
- NEVER add Co-Authored-By lines to commits
- NEVER use git rebase
- NEVER use git push --force or git push -f

**Pull request rules:**
- ALWAYS open pull requests against the repository's default branch (`dev`)

**Code review rules:**
- NEVER submit AI-generated review comments (PR reviews, inline comments, or issue comments) to GitHub automatically
- ALWAYS present review findings to the human reviewer for triage first, and only post the ones they explicitly approve, after they explicitly ask for them to be posted

**Workspace crate rules:**
- When adding, splitting out, or renaming a workspace member crate, ALWAYS classify it in `scripts/coverage.sh`: add it to `COV_PACKAGES` (measured) or `COV_SKIP_PACKAGES` (skipped, with a reason comment). CI's coverage job fails on any unclassified member.

**Cargo dependency rules:**
- ALWAYS declare external dependencies in the root `[workspace.dependencies]` and inherit them in crates via `{ workspace = true }`. Never add inline-versioned dependencies (e.g. `foo = "1.2"`) to a crate's `Cargo.toml`.
- On the inheriting line you may only add `features` (additive) and `optional`; per Cargo, `version` and `default-features` cannot appear there, so set `default-features` in the workspace declaration (e.g. `hex = { version = "0.4", default-features = false }`).

**Automatic formatting:**
- ALWAYS run `/format` after generating or modifying Rust code
- ALWAYS run `/format` before creating any git commit
- This ensures all code follows project formatting standards (Rust, TOML, feature propagation) and passes clippy

**Design & spec discipline:**
- `docs/design/` is the **canonical, review-gated source of truth** (enforced by `.github/CODEOWNERS`). Reason and implement *from* it; treat it as the spec.
- **Validate code against the design.** When writing or changing code, check it against `docs/design/`. On any divergence, **stop and flag** — don't proceed on assumptions.
- **Prefer flagging over quietly editing the design to match the code.** If implementation and design disagree, treat it as a *finding*: open or reference an issue and discuss before changing the spec, rather than silently reconciling the gap.
- If something in the design looks **wrong or vulnerable**, **flag and discuss** (open an issue and ping the design owner) — don't just fix it. Changes to the design itself go through a PR reviewed per `.github/CODEOWNERS`.
- **`docs/reference/`** is *derived* documentation and **not** gated. When you change behavior, **update the relevant `reference/` doc** so it keeps reflecting the implementation.
- **`docs/drafts/`** is unratified / WIP — don't treat it as authoritative or reason from it as if it were the spec.

## Project Overview

Scalable Web3 Storage is a decentralized storage system built on Substrate with game-theoretic guarantees. Storage providers lock stake and face slashing for data loss, while the chain acts as a credible threat rather than the hot path.

**Architecture**: Two-node system where blockchain handles accountability and provider nodes handle actual storage:
- **Parachain Node**: On-chain logic for stake, agreements, checkpoints, and challenges
- **Provider Node**: Off-chain HTTP server for data upload, download, and MMR commitment

**Key Purpose**: Enable trustless storage where normal operations (reads, writes) happen off-chain via HTTP, and the chain is only touched for setup, checkpoints, and disputes.

## Build Commands

```bash
# Build everything (release)
cargo build --release

# Build specific components
cargo build --release -p storage-parachain-runtime
cargo build --release -p pallet-storage-provider
cargo build --release -p storage-provider-node
cargo build --release -p storage-client

# Build with runtime benchmarks
cargo build --release --features runtime-benchmarks

# Using just (recommended)
just build
```

## Test Commands

```bash
# Run all tests
cargo test

# Run pallet tests
cargo test -p pallet-storage-provider

# Run provider node tests
cargo test -p storage-provider-node

# Run client SDK tests
cargo test -p storage-client

# Run file system tests (Layer 1)
cargo test -p file-system-primitives
cargo test -p pallet-drive-registry
cargo test -p file-system-client

# Or test all file system components at once
just fs-test-all

# Run integration tests (require chain + provider already running)
just start-chain     # Terminal 1
just start-provider  # Terminal 2
just demo            # Terminal 3 — Layer-0 PAPI flow
just fs-demo-ci      # Terminal 3 — Layer-1 file-system flow
just s3-demo-ci      # Terminal 3 — Layer-1 S3 flow

# Clippy linting
cargo clippy --all-targets --all-features --workspace -- -D warnings
```

## Formatting

```bash
# Rust formatting (requires nightly)
cargo +nightly fmt --all

# TOML formatting
taplo format --check --config .config/taplo.toml

# Feature propagation lint (checks Cargo.toml feature gates)
zepter run --config .config/zepter.yaml
```

## Run Commands

```bash
# One-time setup (downloads binaries, builds project)
just setup

# Start blockchain
just start-chain

# Start provider node manually
just start-provider

# Check provider health
just health

# Check chain health (relay + parachain + current block)
bash scripts/check-chain.sh


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paritytech/web3-storage](https://github.com/paritytech/web3-storage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
