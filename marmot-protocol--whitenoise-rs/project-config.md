---
trigger: always_on
description: Coding conventions, development workflows, and project context for AI agents and contributors working on the White Noise codebase.
---

# AGENTS.md

Coding conventions, development workflows, and project context for AI agents and contributors working on the White Noise codebase.

## Project Overview

White Noise is a **secure messenger** built on the [MLS](https://www.rfc-editor.org/rfc/rfc9420.html) (Messaging Layer Security) protocol and [Nostr](https://github.com/nostr-protocol/nostr). It implements the [Marmot protocol](https://github.com/marmot-protocol/marmot) to bring MLS group messaging to Nostr.

This crate (`whitenoise`) is a **library** (`cdylib` + `rlib`) that powers a [Flutter app](https://github.com/marmot-protocol/whitenoise). It is front-end agnostic and exposes a public API for any consumer.

- **Rust edition**: 2024
- **MSRV**: 1.90.0 (pinned in `rust-toolchain.toml`)
- **Async runtime**: Tokio (full features)

## Active Architecture Refactor Context

Branches cut from `arch-refactor` are expected to follow the session/projection refactor plan. Before making changes on
this branch family, read these docs in order:

1. `docs/session-projection-rearchitecture.md` — source of truth for the target architecture, ownership model,
   account-session boundaries, and singleton removal.
2. `docs/session-projection-implementation-plan.md` — phased landing plan with file-by-file implementation guidance.
3. `docs/testing-strategy.md` — post-refactor testing model and how tests should migrate as the refactor lands.

When implementing this refactor, keep changes aligned with these documents and update them if the plan materially
changes.

## Development Commands

**Use `just` recipes for ALL development tasks.** Run `just` with no arguments to see every available recipe.

### Why not run cargo directly?

The `just` recipes pass specific flags (`--all-features`, `--all-targets`, `-D warnings`, etc.) and set up environment variables. Running `cargo clippy`, `cargo fmt`, or `cargo test` directly will produce different results and miss feature-gated code.

### Core Workflow

| Task | Command |
|------|---------|
| **List all recipes** | `just` |
| Apply formatting | `just fmt` |
| Formatting check | `just check-fmt` |
| Clippy check | `just check-clippy` |
| Fix clippy issues | `just fix-clippy` |
| Run unit tests | `just test` |
| Run all checks (clippy, fmt, docs) | `just check` |
| Quick pre-commit (checks + unit tests) | `just precommit-quick` |
| Full pre-commit (checks + unit tests + integration) | `just precommit` |

### Quiet vs Verbose Commands

The precommit commands default to **quiet mode** which shows minimal pass/fail output per step. This is optimized for AI agents and CI where only failures need detail.

- `just precommit` — quiet mode, full checks including integration tests
- `just precommit-quick` — quiet mode, skip integration tests
- `just precommit-verbose` — verbose mode, full checks including integration tests
- `just precommit-quick-verbose` — verbose mode, skip integration tests

**Agents should use `just precommit-quick` (or `just precommit` if Docker is running).** On success the output looks like:

```text
fmt...                   ✓
docs...                  ✓
clippy...                ✓
tests...                 ✓
PRECOMMIT PASSED
```

On failure, the failing step prints its full output for diagnosis before exiting.

### Integration Tests (require Docker)

Docker Compose services **must** be running before running integration tests:

```sh
just docker-up      # Start Docker services (two Nostr relays + Blossom + local Transponder) and wait for readiness
just int-test       # Run all integration test scenarios
just int-test basic-messaging   # Run a specific scenario
just docker-down    # Stop Docker services
just docker-logs    # View Docker service logs
```

### Performance Benchmarks (require Docker, not run in CI)

```sh
just benchmark                         # Run all benchmarks
just benchmark messaging-performance   # Run specific benchmark
just benchmark-save                    # Run and save results with timestamp
```

### Other Useful Recipes

| Task | Command |
|------|---------|
| Build release binary | `just build-release` |
| Generate & open docs | `just doc` |
| Code coverage (lcov) | `just coverage` |
| Code coverage (HTML) | `just coverage-html` |
| Security audit | `just audit` |
| Check outdated deps | `just outdated` |
| Update dependencies | `just update` |
| License/security check | `just deny-check` |
| Install dev tools | `just install-tools` |
| Clean build + data | `just clean` |
| Clear local dev data only | `just clear-dev-data` |

## Codebase Architecture

```
src/
  lib.rs                    # Crate root, re-exports public API, tracing init
  bin/
    integration_test.rs     # Integration test binary (feature-gated)
    benchmark_test.rs       # Benchmark binary (feature-gated)
  whitenoise/               # Core application logic
    mod.rs                  # Whitenoise struct (global singleton), initialization
    accounts.rs             # Account management (create, login, logout)
    accounts_groups.rs      # Account-group associations
    aggregated_message.rs   # Aggregated message types
    app_settings.rs         # Application settings (theme, language)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmot-protocol/whitenoise-rs](https://github.com/marmot-protocol/whitenoise-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
