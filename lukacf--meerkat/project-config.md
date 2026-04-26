---
trigger: always_on
description: This file provides guidance to agents when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to agents when working with code in this repository.
Whatever you do, remember: All hail Clippy. Clippy sees all, knows all, and tolerates nothing.

## Project Overview

Meerkat (`rkat`) is a minimal, high-performance agent harness for LLM-powered applications written in Rust. It provides the core execution loop for agents without opinions about prompts, tools, or output formatting.

**Naming convention:**
- Project/branding: **Meerkat**
- CLI binary: **rkat**
- Crate names: `meerkat`, `meerkat-core`, `meerkat-client`, etc.
- Config directory: `.rkat/`
- Environment variables: API keys only (RKAT_* secrets and provider-native keys)

## Build and Test Commands

```bash
# Build everything
./scripts/repo-cargo build --workspace

# Run fast tests (unit + integration-fast; skips doctests)
./scripts/repo-cargo nextest run --workspace --show-progress none --status-level none --final-status-level fail

# Run all tests including doc-tests (SLOW due to doc-test compilation)
./scripts/repo-cargo test --workspace

# Run deterministic end-to-end lane
./scripts/repo-cargo e2e-fast

# Run explicit build/composition end-to-end lane (ignored by default)
./scripts/repo-cargo test -p meerkat-integration-tests --test e2e_build_lane -- --ignored

# Run real local-resource end-to-end lane
./scripts/repo-cargo e2e-system

# Run targeted live-provider lane (ignored by default)
./scripts/repo-cargo e2e-live

# Run kitchen-sink live smoke lane (ignored by default)
./scripts/repo-cargo e2e-smoke

# Run per-model catalog validation lane (ignored by default)
./scripts/repo-cargo e2e-models

# Cargo aliases (defined in .cargo/config.toml)
./scripts/cargo-rct       # Fast tests (unit + integration-fast)
./scripts/repo-cargo unit # Unit tests only
./scripts/repo-cargo int  # Integration-fast tests only
./scripts/repo-cargo e2e-fast    # Deterministic e2e lane
./scripts/repo-cargo test -p meerkat-integration-tests --test e2e_build_lane -- --ignored  # Build/composition lane
./scripts/repo-cargo e2e-system  # Real binary / local resource lane
./scripts/repo-cargo e2e-live    # Targeted live-provider lane
./scripts/repo-cargo e2e-smoke   # Compound live-provider smoke lane
./scripts/repo-cargo e2e-models  # Live per-model catalog validation (on-demand / pre-release)

# Legacy compatibility shims (during migration)
./scripts/repo-cargo int-real  # Alias for e2e-system
./scripts/repo-cargo e2e       # Alias for e2e-live + e2e-smoke

# Run the CLI
./scripts/repo-cargo run -p meerkat-cli -- run "prompt"

# Run a specific example
ANTHROPIC_API_KEY=... ./scripts/repo-cargo run --example simple
```

## Build Efficiency

This is a large workspace (~25 crates). Careless builds waste minutes. Follow these rules:

**Use package-scoped commands during development.** Do NOT default to `--workspace` for every build or check. Scope to the crate you're changing and its immediate dependents:

```bash
# Editing meerkat-core? Check just what you touched + direct dependents
./scripts/repo-cargo check -p meerkat-core -p meerkat-runtime -p meerkat-session

# Editing meerkat-mob? Check the mob subtree
./scripts/repo-cargo check -p meerkat-mob -p meerkat-mob-mcp

# Running tests for one crate
./scripts/repo-cargo nextest run -p meerkat-mob

# Only use workspace-wide commands for final verification
./scripts/repo-cargo clippy --workspace -- -D warnings
./scripts/repo-cargo nextest run --workspace --status-level none --final-status-level fail
```

**Never run parallel cargo commands.** They deadlock on the workspace file lock. Run builds sequentially.

**Always use `./scripts/repo-cargo`** instead of bare `cargo`. The wrapper manages per-worktree build caches and avoids cross-worktree cache pollution.

**Key dependency chains to know** (touching a crate rebuilds everything downstream):
- `meerkat-core` → rebuilds almost everything (~27s incremental)
- `meerkat-runtime` → rebuilds mob, rpc, rest, cli, integration tests
- `meerkat-mob` → rebuilds mob-mcp, rpc, rest, cli, integration tests
- Leaf crates (`meerkat-models`, `meerkat-machine-schema`) → fast, minimal cascade

## Architecture

```
meerkat-models    → Curated model catalog and provider profile rules (leaf crate, no meerkat deps)
                     Single source of truth for model defaults, allowlists, capability detection,
                     and parameter schemas. Consumed by core, client, tools, and facade.
meerkat-core      → Agent loop, types, budget, retry, state machine (no I/O deps)
                     Also: SessionService trait, Compactor trait, MemoryStore trait, SessionError
meerkat-client    → LLM provider clients (Anthropic, OpenAI, Gemini) implementing AgentLlmClient
meerkat-providers → Provider runtime registry + OAuth + TokenStore + cloud authorizers (AWS/GCP/Azure)
                     Owns `ProviderRuntimeRegistry`, `ResolverEnvironment`, and the typed
                     `{backend_kind, auth_method}` matrix per provider. Consumed by the facade
                     and surface crates for realm-scoped credential resolution.
meerkat-store     → Session persistence (SqliteSessionStore, JsonlStore, MemoryStore) implementing SessionStore
meerkat-tools     → Tool registry and validation implementing AgentToolDispatcher

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukacf/meerkat](https://github.com/lukacf/meerkat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
