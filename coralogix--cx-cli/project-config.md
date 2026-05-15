---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`cx` - a Rust CLI for querying Coralogix observability data (logs, metrics, traces, dashboards, alerts) from the terminal. Supports multi-profile fan-out, multiple output formats (text/json/agents), and AI-optimized result spilling.

## Build & Development

```bash
cargo build                         # Dev build
cargo build --release               # Release build (stripped, LTO)
cargo fmt                           # Format code
cargo clippy                        # Lint
cargo test                          # Run all tests
cargo test <test_name>              # Run a single test
cargo test -- --ignored             # Run integration tests (filesystem-dependent)
cargo test --test e2e -- --ignored --test-threads=1   # E2E vs. Coralogix test team (needs CX_API_KEY)
cargo run -- <args>                 # Run CLI in dev mode
```

Rust toolchain is pinned to **1.94.1** via `rust-toolchain.toml`.

## Command Hierarchy

The CLI is organized into 26 commands grouped by domain. `cx --help` shows this layout:

```
Query:
  logs               Query logs using DataPrime syntax
  spans              Query spans using DataPrime syntax
  metrics            Query metrics using PromQL
  dataprime          DataPrime language reference and raw queries
  search-fields      Search log/span fields semantically

Observe:
  dashboards         Manage dashboards and dashboard folders
  views              Manage saved views and view folders
  slos               Manage SLO definitions

Detect & Respond:
  alerts             Manage alert definitions and suppression rules
  incidents          Manage and triage incidents

Notifications:
  notifications      Manage connectors, routers, presets, and notification testing
  webhooks           Manage outgoing webhooks and automation actions

Data Pipeline:
  parsing-rules      Manage log parsing rules
  enrichments        Manage enrichment rules and custom enrichment tables
  e2m                Manage Events2Metrics definitions
  recording-rules    Manage Prometheus recording rule groups

Cost & Storage:
  usage              View data usage and consumption metrics
  tco                Manage TCO policies and settings
  retentions         Manage data retention settings
  archive (risky)    Manage data archive storage configuration

Integrations:
  integrations       Manage integrations, extensions, and contextual data

Access:
  iam (risky)        Manage API keys, roles, scopes, users, groups, and IP access

Agent:
  schema             Output the full command tree as JSON for agent consumption

Local:
  profiles           Manage profiles (list, add, delete, set-default)
  cleanup            Remove stale temp files
```

**Agent discovery:** `cx schema` outputs the full command tree (commands, subcommands, flags, descriptions) as JSON. Agents should call `cx schema` to discover available commands rather than parsing help text.

**Key renames from prior flat layout:**
- `rule-groups` -> `parsing-rules`
- `tco-policies` -> `tco`
- `data-usage` -> `usage`
- `data-archive` -> `archive`

**Wrapper commands** (merged from multiple former top-level commands):
- `notifications` = connectors + routers + presets + notification-test
- `webhooks` = outgoing webhooks + actions (formerly `actions`)
- `enrichments` = enrichment rules + custom enrichment tables (formerly `custom-enrichments`)
- `integrations` = extensions + contextual-data
- `iam` = api-keys + roles + scopes + users + team-groups + ip-access

**Risky commands:** `iam` and `archive` are marked `(risky)` in help output. All write operations (create, update, delete, enable, disable, set, set-status) under these commands require interactive confirmation. Pass `--yes` to skip the prompt (e.g., in scripts or CI). Non-interactive terminals without `--yes` get a clear error. The confirmation logic lives in `src/safety.rs`.

## Architecture

### Execution Flow

1. **CLI parsing** (`main.rs`) - Clap derive macros define the command tree
2. **Config resolution** (`config.rs`) - Loads `~/.cx/config.toml` + `~/.cx/profiles/*.toml`, resolves per-profile credentials and region endpoints
3. **Target building** (`execution.rs`) - Each profile becomes an `ExecutionTarget` wrapping a `ResolvedConfig` + `CxClient`
4. **Fan-out** (`execution.rs::fan_out`) - Runs the command handler concurrently across all targets
5. **Result merging** (`execution.rs::merge_tagged_results`) - Combines per-profile results, tags rows with profile names when multi-profile
6. **Output rendering** (`render.rs`) - Shared helpers for text tables, JSON, and TOON-encoded agents format
7. **Spilling** (`spill.rs`) - If output exceeds `max_dataprime_direct_output_size` (default 100KiB), writes to a temp file and returns the path

### Layout


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coralogix/cx-cli](https://github.com/coralogix/cx-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
