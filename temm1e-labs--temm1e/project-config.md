---
trigger: always_on
description: TEMM1E is a cloud-native Rust AI agent runtime. It connects to messaging channels (Telegram, Discord, WhatsApp, Slack, CLI), routes messages through an agent loop that calls AI providers (Anthropic, OpenAI-compatible), executes tools (shell, browser, file ops), and persists conversation history to memory backends (SQLite, Markdown).
---

# TEMM1E -- Claude Code Project Guide

## Project overview

TEMM1E is a cloud-native Rust AI agent runtime. It connects to messaging channels (Telegram, Discord, WhatsApp, Slack, CLI), routes messages through an agent loop that calls AI providers (Anthropic, OpenAI-compatible), executes tools (shell, browser, file ops), and persists conversation history to memory backends (SQLite, Markdown).

The codebase is a Cargo workspace with 24 crates plus a root `temm1e` binary plus a separate `temm1e-watchdog` supervisor binary (25 crates total).

## Build commands

```bash
# Quick compilation check (fastest feedback loop)
cargo check --workspace

# Build all crates in debug mode
cargo build --workspace

# Run all tests
cargo test --workspace

# Run tests for a specific crate
cargo test -p temm1e-<crate>

# Clippy lints (CI gate -- treats warnings as errors)
cargo clippy --workspace --all-targets --all-features -- -D warnings

# Format check
cargo fmt --all -- --check

# Format code
cargo fmt --all

# Build release binary
cargo build --release --bin temm1e

# Build with TUI feature
cargo build --release --features tui

# Run the binary
cargo run -- start
cargo run -- chat
cargo run --features tui -- tui    # Interactive TUI
cargo run -- status
cargo run -- config validate
```

## Architecture

### Workspace structure

```
crates/
  temm1e-core        -- Shared traits, types, error enum, config loader
  temm1e-gateway     -- HTTP/WebSocket server, routing, session management
  temm1e-agent       -- Agent runtime loop, context, executor
  temm1e-providers   -- AI provider integrations (Anthropic, OpenAI-compatible)
  temm1e-codex-oauth -- ChatGPT Plus/Pro via OAuth PKCE
  temm1e-tui         -- Interactive terminal UI (ratatui, syntect, crossterm)
  temm1e-channels    -- Messaging channels (CLI, Telegram, Discord, WhatsApp Web, WhatsApp Cloud API, Slack)
  temm1e-memory      -- Persistent memory backends (SQLite, Markdown)
  temm1e-tools       -- Agent tool implementations (shell, browser, Prowl, file ops)
    browser_session.rs     -- OTK interactive login with annotated screenshots
    browser_observation.rs -- Layered observation (tree → DOM → screenshot)
    browser_pool.rs        -- Lock-free browser context pool for swarm browsing
    credential_scrub.rs    -- Credential scrubber (LLM context isolation)
    prowl_blueprints.rs    -- Web-specific blueprints (login, search, extract, compare)
    prowl_blueprints/login_registry.rs -- 100+ service login URL registry
  temm1e-vault       -- Secret storage with ChaCha20-Poly1305 encryption
  temm1e-skills      -- Skill registry and execution
  temm1e-hive        -- Many Tems: swarm intelligence, pack coordination, scent field
  temm1e-distill     -- Eigen-Tune: self-tuning distillation engine (runtime-gated by [eigentune] enabled=true; local serving requires the second opt-in enable_local_routing=true; see tems_lab/eigen/LOCAL_ROUTING_SAFETY.md for the seven-gate safety chain)
  temm1e-gaze        -- Tem Gaze: desktop vision control (xcap + enigo), SoM overlay
  temm1e-perpetuum   -- Perpetuum: perpetual time-aware entity, scheduling, monitors, volition
  temm1e-anima       -- Tem Anima: emotional intelligence, user profiling, personality system
  temm1e-cores       -- TemDOS: specialist sub-agent cores (architecture, code-review, test, debug, web, desktop, research, creative)
  temm1e-cambium     -- Cambium: gap-driven self-grow (zone_checker, trust, budget, history, sandbox, pipeline, deploy)
  temm1e-watchdog    -- Immutable supervisor binary that monitors temm1e PID and restarts on crash. Part of Cambium's immutable kernel. Also hosts the Witness Root Anchor: periodically reads the live Ledger root hash and writes a sealed (chmod 0400) copy that the main process cross-checks for tamper detection.
  temm1e-witness     -- Witness verification system: pre-committed Oaths, hash-chained tamper-evident Ledger, three-tier predicate verifier (deterministic Tier 0 + LLM-backed Tier 1/2). Wires into AgentRuntime via with_witness(); enforces the Five Laws (Pre-Commitment, Independent Verdict, Immutable History, Loud Failure, Narrative-Only FAIL); validated end-to-end against gpt-5.4 + Gemini 3 Flash Preview on real Tem source-code refactors. See tems_lab/witness/RESEARCH_PAPER.md for the theory and tems_lab/witness/EXPERIMENT_REPORT.md for empirical results.
  temm1e-mcp         -- MCP client (stdio + HTTP, 14-server registry)
  temm1e-automation  -- Cron jobs and scheduled tasks
  temm1e-observable  -- OpenTelemetry tracing and metrics
  temm1e-filestore   -- File storage (local, S3)
  temm1e-test-utils  -- Shared test utilities
src/
  main.rs             -- CLI entry point (clap)
```

### Architecture rules

1. **Traits in core, implementations in crates**: All shared traits (`Channel`, `Provider`, `Memory`, `Tool`, `FileTransfer`, etc.) are defined in `temm1e-core/src/traits/`. Implementations go in their respective crates.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [temm1e-labs/temm1e](https://github.com/temm1e-labs/temm1e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
