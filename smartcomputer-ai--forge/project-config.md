---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Note: CLAUDE.md is a symlink to AGENTS.md — they are the same file.

## Build & Test Commands

```bash
cargo build                                    # Build entire workspace
cargo test                                     # Run all workspace tests
cargo test -p forge-llm                        # Test a single crate
cargo test -p forge-agent                      # Test agent crate
cargo test -p forge-attractor --tests          # Test attractor (integration tests only)
cargo test -p forge-cli --tests                # Test CLI (integration tests only)
cargo test -p forge-cxdb-runtime               # Test CXDB runtime
cargo test -p forge-llm test_name              # Run a single test by name
cargo test -p forge-llm -- --nocapture         # Run tests with stdout visible
```

Opt-in infrastructure tests (ignored by default, require local services or CLIs):
```bash
cargo test -p forge-cxdb-runtime --test live -- --ignored      # needs running CXDB server
cargo test -p cxdb --test integration -- --ignored             # needs running CXDB server
cargo test -p forge-llm --test cli_agent_e2e -- --ignored      # needs claude/codex/gemini CLIs (OAuth, no API keys)
cargo test -p forge-cli --test e2e_pipeline -- --ignored        # full-stack e2e: DOT → CLI agent → JSONL → artifacts → CXDB
```

Live provider tests (ignored by default, require API keys — costs real money):
```bash
cargo test -p forge-llm --test openai_live -- --ignored        # needs OPENAI_API_KEY
cargo test -p forge-llm --test anthropic_live -- --ignored      # needs ANTHROPIC_API_KEY
cargo test -p forge-agent --test openai_live -- --ignored       # needs OPENAI_API_KEY
cargo test -p forge-agent --test anthropic_live -- --ignored    # needs ANTHROPIC_API_KEY
cargo test -p forge-attractor --test live -- --ignored          # needs OPENAI_API_KEY
```

CLI host usage:
```bash
cargo run -p forge-cli -- run --dot-file examples/01-linear-foundation.dot --backend mock
cargo run -p forge-cli -- resume --dot-file <FILE> --checkpoint <PATH> --backend mock
cargo run -p forge-cli -- inspect-checkpoint --checkpoint <PATH> --json
```

## Architecture

Forge is a spec-first software factory stack centered on Attractor-style DOT pipeline orchestration. Crate dependency graph (bottom-up):

```
forge-cxdb            vendored CXDB client SDK (binary protocol, TLS, reconnect)
    ↓
forge-cxdb-runtime    CXDB runtime integration (typed store, client traits, testing fakes)
    ↓
forge-llm             unified multi-provider LLM client (OpenAI + Anthropic adapters)
    ↓
forge-agent           coding agent loop (session state machine, tools, provider profiles)
    ↓
forge-attractor       DOT pipeline parser → graph IR → execution engine + handlers
    ↓
forge-cli             CLI binary (clap) — run/resume/inspect Attractor pipelines
```

Key architectural patterns:
- **Trait-based adapters** — `ProviderAdapter` (stateless LLM call), `AgentProvider` (provider-owned agent loop), `NodeHandler` (pipeline nodes), `ExecutionEnvironment` (file/shell), `Interviewer` (HITL), `CxdbBinaryClient`/`CxdbHttpClient` (persistence). Shared via `Arc<dyn Trait>`.
- **Unified agent provider** — Every provider (HTTP API or CLI subprocess) implements `AgentProvider::run_to_completion()`. HTTP providers compose `ProviderAdapter` + `ToolRegistry` + `ExecutionEnvironment`. CLI providers (Claude Code, Codex, Gemini) spawn subprocess and parse JSONL. Session delegates to the provider. See `spec/06-unified-agent-provider-spec.md`.
- **Middleware chain** — LLM client composes middleware in onion model for `complete()`/`stream()`.
- **Explicit provider configuration** — Providers are explicitly configured, not auto-discovered from environment variables.
- **Session state machine** — `SessionState` enum (Idle/Processing/AwaitingInput/Closed) with explicit `can_transition_to()` validation.
- **Hierarchical errors** — Each crate defines its own `thiserror` error enums wrapping child crate errors.
- **Serialization** — JSON for external interfaces, msgpack (`rmp-serde`) for CXDB binary protocol and internal persistence.
- **Async runtime** — `tokio` with `current_thread` flavor everywhere (main and tests).
- **Rust edition 2024** for all Forge crates; vendored `cxdb` uses edition 2021.
- **Centralized status.json** — The runner writes `status.json` for ALL node types (not individual handlers). Fields: `outcome`, `preferred_next_label`, `suggested_next_ids`, `context_updates`, `notes`, `failure_reason`.
- **Model stylesheet specificity** — Selectors: `*` (universal, 0) < shape name (1) < `.class` (2) < `#node_id` (3). Nodes without an explicit `shape` attribute default to `"box"`.
- **Parallel error_policy** — Parallel handler supports `error_policy` attribute: `continue` (default), `fail_fast` (abort on first failure), `ignore` (downgrade failures to success).
- **Goal-gate enforcement** — Engine checks ALL graph nodes with `goal_gate=true` before allowing exit, including nodes never visited during execution.

## Specifications

Primary specs live in `spec/` — these are the source of truth:

- `spec/00-vision.md` — vision + principles + techniques

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smartcomputer-ai/forge](https://github.com/smartcomputer-ai/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
