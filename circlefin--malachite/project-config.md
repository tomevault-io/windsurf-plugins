---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

Malachite is a Byzantine Fault Tolerant (BFT) consensus engine implementing the Tendermint consensus algorithm in Rust. It is a Cargo workspace rooted at `code/Cargo.toml` (not the repo root).

## Build and Test Commands

All commands run from the `code/` directory:

```bash
cargo build                    # Build the workspace
cargo lint                     # Clippy with all features
cargo fmt --check              # Check formatting
cargo nextest run -p <CRATE>   # Run tests for a specific crate
cargo nextest run -p <CRATE> -E 'test(test_name)'  # Run a single test
cargo all-tests                # All tests except MBT
cargo mbt                      # Model-based tests (requires Quint)
```

Integration test packages:
- `arc-malachitebft-test` — test app integration tests
- `arc-malachitebft-discovery-test` — discovery protocol tests

Note: Published crate names use the `arc-malachitebft-*` prefix, but workspace dependency aliases use `malachitebft-*` (e.g., `malachitebft-engine`). When specifying `-p` to cargo, use the published name.

## Architecture

### Core Design: Suspended Effects Pattern

Malachite's consensus logic is **pure and stateless** — it performs no I/O. Instead, it uses a coroutine-based effect system:

1. The `Context` trait (`core-types`) parameterizes all consensus types (Address, Height, Value, Vote, Proposal, ValidatorSet, SigningScheme, etc.)
2. The consensus `handle()` function is a coroutine (via `genawaiter`) that yields **Effects** (sign, publish, decide, WAL append, etc.) and receives **Resume** values back
3. The `process!` macro (`core-consensus/src/macros.rs`) drives this loop: feed an Input → run coroutine → handle yielded Effect → resume with result → repeat
4. **Actors** (via `ractor`) handle effects asynchronously in the engine layer

### Crate Organization

**Core consensus** (pure, no I/O, `no_std`-friendly):
- `core-types` — `Context` trait and all type traits (Value, Vote, Proposal, etc.)
- `core-state-machine` — single-round Tendermint state machine
- `core-votekeeper` — vote aggregation and quorum tracking
- `core-driver` — multi-round orchestration
- `core-consensus` — effect/input system, `process!` macro, top-level handlers

**Engine layer** (async, actor-based):
- `engine` — actor wiring: Node (supervisor), Consensus, Host, Network, WAL, Sync actors
- `app` / `app-channel` — high-level channel-based application interface
- `network` — libp2p-based P2P layer
- `discovery` — peer discovery protocol
- `sync` — block synchronization protocol
- `wal` — write-ahead log for crash recovery
- `config` / `metrics` / `codec` / `proto` / `peer` / `signing` / `signing-ed25519` / `signing-ecdsa`

**Test:**
- `test/` — integration test app, framework, MBT, CLI, mempool utilities

### Three Integration Levels

1. **Channel-based** (`app-channel`) — highest level, batteries-included (networking, sync, WAL)
2. **Actor-based** (`engine`) — swap individual actors (e.g., custom network layer)
3. **Core library** (`core-consensus`) — fully agnostic, bring your own everything

### Key Patterns

- The `perform!` macro inside effect handlers yields an effect and destructures the resume value in one expression
- `ConsensusState<Ctx>` holds the `Driver<Ctx>` (round state machine), input queue, proposal keeper, and timing metadata
- Message flow: Network/Host/Timer → Consensus Actor → `process!(input, state, on: effect)` → Effect Handler routes to other actors → Resume value fed back

## Conventions

- Commit messages: conventional commits with Jira references, e.g., `feat: Discovery peers request rate limiter`
- Rust edition 2021, MSRV 1.88
- Workspace lints: `clippy::disallowed_types = "deny"` — check `code/clippy.toml` for the disallowed types list

## CI Checks and PR Workflow

### CI checks on pull requests

All checks run automatically on PRs. Checks under `rust.yml` and `quint.yml` are skipped when their respective directories (`code/`, `specs/`) have no changes.

| Workflow | Jobs | Triggers on |
|----------|------|-------------|
| **Rust** (`rust.yml`) | Unit tests, Integration tests (discovery, test app), Clippy, Formatting, no_std compatibility, MSRV, Standalone feature checks | `code/` changes |
| **Quint** (`quint.yml`) | Typecheck, Test | `specs/` changes |
| **Semver** (`semver.yml`) | Detect semver violations (posts PR comment if breaking) | `code/` changes |
| **Coverage** (`coverage.yml`) | Code coverage report (posted as PR comment) | `code/` or `specs/` changes |
| **Spelling** (`typos.yml`) | Typo check via `typos` | Always |
| **PR** (`pr.yml`) | PR title lint (conventional commits) | Always |
| **AI PR** (`ai-pr.yml`) | Automated Claude Code review on open/sync; PR notes on `ai-pr-notes` label | Always |

### Before opening a PR

Run these from the `code/` directory to catch issues locally:

```bash
cargo fmt --all              # Fix formatting
cargo lint                   # Clippy with all features
cargo all-tests              # All tests except MBT
```

If you changed specs, also run from the repo root:

```bash
cargo mbt                    # Model-based tests (requires Quint)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [circlefin/malachite](https://github.com/circlefin/malachite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
