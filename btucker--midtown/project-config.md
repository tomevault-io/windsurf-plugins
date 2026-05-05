---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Midtown is a multi-Claude Code workspace manager. It coordinates a Lead (human-facing Claude Code in a terminal session) and multiple autonomous Coworkers (headless Claude Code sessions), each running in isolated git worktrees. Communication happens through an IRC-style append-only channel log.

See [docs/architecture.md](docs/architecture.md) for a full architecture reference.

## Build & Development Commands

```bash
# Build & install
cargo build                     # debug build (daemon only)
cargo install --path .          # release build + install to ~/.cargo/bin/

# Test
cargo test                      # unit + non-ignored integration tests
cargo test <test_name>          # run a single test by name
cargo test --test daemon_e2e -- --ignored --test-threads=1  # E2E tests

# Lint (CI enforces -D warnings)
cargo clippy --all-targets --all-features -- -D warnings
cargo fmt --all -- --check

# Code coverage (requires: cargo install cargo-llvm-cov)
./scripts/coverage.sh           # HTML report → target/llvm-cov/html/
./scripts/coverage.sh --text    # text summary
./scripts/coverage.sh --open    # HTML report and open in browser

```

**Shared build cache (sccache)**: Midtown uses many worktrees simultaneously. sccache shares compiled dependency artifacts across all of them, so new worktrees build in ~15-30s instead of ~2-4min (only the `midtown` crate recompiles).

**One-time setup** (modifies `~/.cargo/config.toml`, which applies globally to all Rust projects on this machine):
```bash
cargo install sccache

# Add to ~/.cargo/config.toml:
# [build]
# rustc-wrapper = "sccache"
# [env]
# CARGO_INCREMENTAL = "0"
# SCCACHE_DIR = "/Users/<you>/.midtown/sccache"  # absolute path required (~ not expanded in TOML)

sccache --show-stats   # verify cache hits after a second build
```

**Test file placement**: Put unit tests in separate files (`src/daemon/pr_tests.rs`) rather than inline `#[cfg(test)] mod tests` blocks. Use `#[path = "pr_tests.rs"] #[cfg(test)] mod tests;` in the source file to maintain private access. This keeps PR diffs focused — reviewers can see how much is test vs. implementation at a glance. Integration/E2E tests go in `tests/` as usual.

**Pre-commit hooks** (cargo-husky): `cargo fmt`, `cargo clippy`, and `biome check` (web-app) run automatically on commit. If any check fails, the commit is rejected — fix before retrying. The biome check only runs when `web-app/node_modules` is installed, so pure Rust worktrees are unaffected.

**E2E tests** run with `--ignored`. CI uses `MIDTOWN_WEBHOOK_PORT=0` and `MIDTOWN_CHAT_MONITOR=0` to disable network features during testing.

**Containerized E2E tests** (canonical way to run E2E — reproducible environment):

```bash
# Using the CLI:
midtown e2e auth                            # one-time: authenticate for container testing
midtown e2e run coordination                # fast, no auth needed
midtown e2e run full                        # real Claude, needs auth setup first

# Or use the scripts directly:
./scripts/e2e-container.sh coordination
./scripts/e2e-container.sh full
```

**While waiting for GitHub CI**: After pushing a PR, don't wait idle for CI results. Run the full containerized E2E tests locally:

```bash
midtown e2e run coordination    # run while CI is in progress
```

This catches failures faster than waiting for GitHub Actions and keeps you productive. The container environment matches CI, so local passes should match remote passes.

**End-of-work cleanup**: run `cargo clean` after completing work to reclaim disk space from build artifacts.

## Daemon V2 Architecture

The daemon uses an event-sourced architecture (`src/daemon_v2/`). The v1 daemon has been removed. See [docs/v2-architecture.md](docs/v2-architecture.md) for the full architecture and [docs/v2-spec.md](docs/v2-spec.md) for the user-facing spec.

### Core pipeline

```
Command  →  executor (I/O)  →  DomainEvent(s)  →  update projections
   ↑                                                       |
   +──── decision functions read projections (immutable) ──+
```

### Decision functions are pure

Decision functions in `src/daemon_v2/decisions/` take `&Projections` and return `Vec<Command>`. No I/O, no mutation, no async. The executor in `src/daemon_v2/executor/` handles all side effects.

### Projections replace tick fields

V1 uses 50+ `tick_*` fields populated by `prepare_tick()`. V2 uses three auto-maintained projections:
- `AgentIndex` — agents by id/name/task/channel/thread, running set
- `WorkIndex` — tasks and PRs with pre-indexed views
- `ChannelIndex` — channel metadata and settings
- `CooldownTracker` — unified cooldown mechanism (replaces 10 ad-hoc ones)

### Nudge routing

All nudge routing goes through `chat::route_message()`. Rules:
1. Thread-bound agent gets thread replies. No thread binding → channel lead gets them.
2. Channel lead gets all top-level messages.
3. @mentions and !N task refs nudge the named/assigned agent.
4. **No running-state checks** — if the target is stopped, the executor resumes it.
5. Self-nudges are suppressed.

### Key conventions for v2


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btucker/midtown](https://github.com/btucker/midtown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
