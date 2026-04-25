---
trigger: always_on
description: Manages connections to multiple Kubernetes clusters:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Priority #1: CORRECTNESS

**Correctness is our number one priority.** No hacks, no workarounds, no shortcuts.

- Code must be maximally correct - if something feels wrong, it probably is
- Match established semantics (Kubernetes, SQL/PostgreSQL, Rust idioms)
- Special cases are acceptable ONLY when they reflect actual underlying semantics (e.g., ConfigMaps have `data` not `spec` because that's how K8s works)
- NEVER assume or decide to take shortcuts without consulting me first
- Any deviations from standards, best practices, or established patterns must be discussed before implementation
- Anything that would trigger the "spidey senses" of a very senior engineer/architect should make you pause and ask
- When in doubt, ask - do not proceed with assumptions

## Priority #2: Performance

Performance is our second priority - still very important.

- Minimize API calls to Kubernetes (push filters to API when possible)
- Parallel queries across clusters
- Efficient data structures and algorithms
- But never sacrifice correctness for performance

## Git Commits

- Do NOT add Claude attributions to commit messages (no "Generated with Claude Code", no "Co-Authored-By: Claude")
- Write clear, conventional commit messages focusing on what changed and why
- Always run `cargo fmt` before committing
- Always verify with `cargo clippy` for lint issues before committing

## Code Navigation

- Use the Serena MCP with rust-analyzer for code navigation and symbol lookup
- Prefer symbolic tools (find_symbol, find_referencing_symbols, get_symbols_overview) over grep for Rust code

## Project Overview

k8sql exposes the Kubernetes API as a SQL-compatible database using Apache DataFusion as the query engine. kubectl contexts are treated as databases (switchable with `USE cluster1;`) and Kubernetes resources are exposed as tables. The `_cluster` column is part of every table's primary key, enabling cross-cluster queries.

k8sql supports both SQL and [PRQL](https://prql-lang.org) (Pipelined Relational Query Language) with automatic detection - just write either language and it works.

## Build Commands

```bash
cargo build              # Debug build
cargo build --release    # Release build
cargo test               # Run all tests
```

## Usage Examples

```bash
# Interactive REPL (default)
k8sql

# Batch mode - SQL queries
k8sql -q "SELECT * FROM pods"
k8sql -q "SELECT name, status->>'phase' as phase FROM pods WHERE namespace = 'kube-system'"

# Batch mode - PRQL queries (auto-detected)
k8sql -q "from pods | take 5"
k8sql -q "from pods | filter namespace == 'kube-system' | select {name, namespace} | take 10"

# Multi-cluster queries
k8sql -q "SELECT * FROM pods WHERE _cluster = 'prod'"
k8sql -q "SELECT * FROM pods WHERE _cluster = '*'"  # All clusters

# Batch mode with context and output format
k8sql -c prod-cluster -q "SELECT * FROM deployments" -o json

# Daemon mode (PostgreSQL wire protocol)
k8sql daemon --port 15432
```

## Architecture

```
src/
├── main.rs                    # Entry point, CLI argument parsing, mode dispatch
├── cli/
│   ├── mod.rs
│   ├── args.rs                # Clap argument definitions
│   └── repl.rs                # Rustyline REPL with completion/highlighting
├── datafusion_integration/
│   ├── mod.rs
│   ├── context.rs             # K8sSessionContext - DataFusion session setup
│   ├── provider.rs            # K8sTableProvider - TableProvider implementation
│   ├── convert.rs             # JSON to Arrow RecordBatch conversion
│   ├── preprocess.rs          # Query preprocessing (PRQL compilation, JSON paths, ->> precedence)
│   ├── prql.rs                # PRQL detection and compilation to SQL
│   ├── json_path.rs           # JSON path syntax preprocessor (.field, [n], [])
│   └── hooks.rs               # Query hooks for filter extraction
├── kubernetes/
│   ├── mod.rs                 # ApiFilters type for filter pushdown
│   ├── cache.rs               # Resource discovery cache persistence
│   ├── client.rs              # K8sClientPool - multi-cluster connection caching
│   ├── discovery.rs           # Dynamic resource discovery and schema generation
│   └── field_selectors.rs     # Field selector registry and types for K8s API pushdown
├── output/
│   ├── mod.rs                 # QueryResult type and format dispatch
│   ├── table.rs               # Pretty table output
│   ├── json.rs                # JSON output
│   ├── csv.rs                 # CSV output
│   └── yaml.rs                # YAML output
└── daemon/
    ├── mod.rs
    └── pgwire_server.rs       # PostgreSQL wire protocol server
```

## Key Components

### DataFusion Integration (`src/datafusion_integration/`)

k8sql uses Apache DataFusion as its SQL query engine:

- **K8sSessionContext** (`context.rs`): Wraps DataFusion's SessionContext, registers all K8s resources as tables, provides JSON functions for querying nested fields.

- **K8sTableProvider** (`provider.rs`): Implements DataFusion's TableProvider trait. Fetches data from Kubernetes API on scan, extracts and pushes filters (namespace, cluster, label selectors) to the API.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ndenev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
