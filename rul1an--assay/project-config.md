---
trigger: always_on
description: Assay is a **Policy-as-Code** engine for Model Context Protocol (MCP) that validates AI agent behavior. It provides deterministic testing (trace replay), runtime security (eBPF/LSM kernel enforcement on Linux), and compliance gates (tool argument/sequence validation).
---

# Assay - AI Agent Context

## What is Assay?

Assay is a **Policy-as-Code** engine for Model Context Protocol (MCP) that validates AI agent behavior. It provides deterministic testing (trace replay), runtime security (eBPF/LSM kernel enforcement on Linux), and compliance gates (tool argument/sequence validation).

## Workspace Structure

Rust monorepo with workspace version `3.31.1` (21 crates). Curated view, grouped by role:

```
crates/
  # Core engine + CLI
  assay-core/       Core evaluation engine (Runner, Store, MCP, Trace, Report, Providers, VCR, Replay Bundle)
  assay-cli/        CLI binary ("assay") - all user-facing commands
  assay-metrics/    Standard metrics (MustContain, RegexMatch, ArgsValid, SequenceValid, etc.)
  assay-common/     Shared types (no_std compatible for eBPF)
  assay-canonical/  Deterministic canonicalization: RFC 8785 (JCS) bytes, sha256 content IDs, semantic digests

  # Evidence + distribution
  assay-evidence/   Evidence bundles (tar.gz with manifest.json + events.ndjson), lint, diff, sanitize
  assay-registry/   Pack Registry client (HTTP, DSSE verification, OIDC auth, local caching, lockfile v2)
  gateway-evidence-replay/  Deterministic replay verifier for gateway-path evidence bundles (standalone)

  # Policy + runtime enforcement
  assay-policy/     Policy compilation (Tier 1: kernel, Tier 2: userspace)
  assay-mcp-server/ MCP server/proxy for runtime policy enforcement (JSON-RPC over stdio)
  assay-monitor/    Runtime eBPF/LSM monitoring (Linux only)
  assay-ebpf/       Kernel eBPF programs (LSM hooks + tracepoints)

  # Measured-run substrate (internal/experimental "Assay-Runner", API unstable)
  assay-runner-core/    Runner orchestration, archive assembly, layer normalizers
  assay-runner-linux/   Linux-only platform adapter, cgroup placement primitives
  assay-runner-schema/  Versioned schema types + constants for Runner v0 contracts

  # Protocol adapters (evidence translation)
  assay-adapter-api/  Adapter API contracts (shared trait surface)
  assay-adapter-a2a/  A2A protocol adapter
  assay-adapter-acp/  ACP protocol adapter
  assay-adapter-ucp/  UCP protocol adapter

  # Simulation + tooling
  assay-sim/        Attack simulation harness (chaos, differential, integrity testing)
  assay-xtask/      Build tooling
assay-python-sdk/   Python SDK (PyO3 bindings + pytest plugin; crate name "assay-it")
```

## Key Commands

```bash
cargo build -p assay-cli                    # Build CLI
cargo test --workspace                      # Run all tests
cargo test -p assay-sim                     # Run sim tests only
cargo clippy --workspace --all-targets -- -D warnings  # Lint
cargo xtask build-ebpf                      # Build eBPF (Linux)
```

## CLI Entry Points

All commands defined in `crates/assay-cli/src/cli/args/mod.rs`, dispatched in `crates/assay-cli/src/cli/commands/mod.rs`. The table below is a representative subset; the CLI has ~40 subcommands (see `commands/` for the full set, including `import`, `project-otel`, `inventory`, `discover`, and the `verify-*` evidence family).

| Command | Purpose | Entry File |
|---------|---------|------------|
| `assay run` | Execute test suite against traces | `commands/mod.rs::cmd_run()` |
| `assay validate` | Stateless policy validation | `commands/validate.rs` |
| `assay sim run` | Attack simulation suite | `commands/sim.rs` |
| `assay evidence lint` | Lint bundles (JSON/SARIF output) | `commands/evidence/lint.rs` |
| `assay evidence diff` | Verified-only bundle comparison | `commands/evidence/diff.rs` |
| `assay evidence explore` | Read-only TUI explorer | `commands/evidence/explore.rs` |
| `assay evidence export` | Export evidence bundles | `commands/evidence.rs` |
| `assay mcp-server` | MCP proxy with policy enforcement | `assay-mcp-server/src/main.rs` |
| `assay monitor` | eBPF runtime monitoring (Linux) | `commands/monitor.rs` |
| `assay sandbox` | Landlock sandbox execution | `commands/sandbox.rs` |
| `assay doctor` | Diagnostic tool | `commands/doctor.rs` |

## Core Architecture

### Execution Flow (CLI -> Core)

```
CLI main.rs -> dispatch() -> build_runner() -> Runner::run_suite()
  Runner creates: Store (SQLite), VcrCache, LLM Client, Metrics, Embedder, Judge, Baseline
  Per test: fingerprint -> cache lookup -> LLM call/replay -> metrics eval -> baseline check -> store
  Output: RunArtifacts -> formatters (console/JSON/JUnit/SARIF)
```

### Key Interfaces

- **`Metric` trait** (`assay-core::metrics_api`): `evaluate(&self, response, expected) -> MetricResult`
- **`LlmClient` trait** (`assay-core::providers::llm`): OpenAI, Fake, Trace replay, Strict wrapper
- **`Embedder` trait** (`assay-core::providers::embedder`): OpenAI, Fake
- **`Store`** (`assay-core::storage`): SQLite wrapper for runs, results, attempts, embeddings
- **`VcrClient`** (`assay-core::vcr`): HTTP record/replay for deterministic LLM testing

### Policy Enforcement (Two-Tier)

- **Tier 1** (Kernel/LSM): Exact paths, CIDRs, ports -> enforced via eBPF in kernel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rul1an/assay](https://github.com/Rul1an/assay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
