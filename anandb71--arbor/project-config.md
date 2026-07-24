---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build
cargo build --workspace

# Test all crates
cargo test --workspace

# Test single crate
cargo test -p arbor-graph
cargo test -p arbor-core

# Test single test by name
cargo test -p arbor-graph -- ranking::tests::test_pagerank_basic

# Lint
cargo clippy --workspace --all-targets --all-features

# Format check
cargo fmt --all -- --check

# Format fix
cargo fmt --all

# Benchmarks (criterion; CI regression gate in .github/workflows/benchmarks.yml)
cargo bench -p arbor-graph

# Release build (CLI binary)
cargo build --locked --release -p arbor-graph-cli

# Run CLI locally
cargo run -p arbor-graph-cli -- <command>
```

## Architecture

Arbor is a **semantic code graph engine** — it parses codebases into a dependency graph and exposes that graph to CLIs, GUIs, WebSocket clients, and AI agents (via MCP).

### Crate Dependency Order

```
arbor-core  →  arbor-graph  →  arbor-watcher
                     │                │
                     └───── arbor-server ─────┐
                                              │
                     arbor-mcp ───────────────┤
                     arbor-cli ───────────────┘
                     arbor-gui ──────────────────→ arbor-{core,graph,watcher}
```

### Crate Roles

**`arbor-core`** — Tree-sitter AST parsing. Extracts functions, classes, structs, imports, and call edges for 9 production languages (Rust, TS/JS, Python, Go, Java, C/C++, C#, Dart) plus 5 fallback parsers. Each language lives in `crates/arbor-core/src/languages/`. `parser_v2.rs` is the active parser; `parser.rs` is legacy.

**`arbor-graph`** — In-memory petgraph + sled persistence. Key modules:
- `builder.rs` — converts parsed nodes/edges into the graph, builds per-file import maps for cross-module edge filtering
- `ranking.rs` — PageRank with 10% weight for test-file callers
- `heuristics.rs` — entry point detection (main, HTTP routes, webhooks, jobs, CLI commands)
- `impact.rs` — blast radius, shortest path (A*)
- `slice.rs` — context trimming (token-aware, tiktoken)
- `symbol_table.rs` — cross-file FQN resolution
- `confidence.rs` — edge confidence scoring
- `store.rs` — sled-backed persistence

**`arbor-watcher`** — `notify`-based file watcher. Debounces at 100ms, respects `.gitignore`, triggers incremental re-parse of changed files only. Two-tier cache: file-level AST + node-level byte ranges.

**`arbor-server`** — Tokio WebSocket server on `ws://localhost:7432`. JSON-RPC methods: `discover`, `impact`, `context`, `graph.subscribe`, `spotlight`. RwLock-protected shared graph state.

**`arbor-mcp`** — MCP bridge for AI agents (stdio by default, stateless HTTP via `arbor bridge --http --port 3333`). Speaks MCP `2026-07-28` with fallback for `2025-03-26` clients. Sixteen tools:
- **Orientation**: `get_map` — ranked, token-budgeted skeleton of the codebase (recommended first call), `get_architecture_overview`
- **Surgical**: `list_entry_points`, `get_callers`, `get_callees`, `search_symbols`, `get_file_graph`, `get_node_detail`, `explain_symbol`
- **Broad**: `get_logic_path`, `get_knowledge_path`, `find_path`, `analyze_impact`, `get_blast_radius` (git-diff based), `audit_security`, `batch_query`

Module layout: `lib.rs` (tool dispatch + `tools/list`), `protocol.rs` (version negotiation, caching metadata), `tasks.rs` (Tasks extension — background indexing returns task handles agents can poll during cold start), `apps.rs` (MCP Apps — interactive blast-radius and architecture-map UIs via `ui://arbor/*` resources), `http.rs` (HTTP transport with `Mcp-Method`/`Mcp-Name` header routing), `git.rs`.

All tools emit a standard JSON envelope: `{ok, tool, arbor_version, data, meta: {node_count, suggested_next_tool, suggested_next_args}}`. Error responses use `{ok: false, error}`. `search_symbols` and `get_map` support pagination (`offset`, `limit`, `hasMore`).

**`arbor-cli`** — Clap CLI with ~30 subcommands. Most command logic lives in `src/commands.rs`; `src/audit/` implements the security audit and `src/hook/` implements agent-harness installation (`arbor hook claude`). Entry point: `src/main.rs`. Dispatches to the other crates. Binary name: `arbor` (crate name: `arbor-graph-cli`).
Key features:
- `map . --exclude-test`: ranked, token-budgeted project skeleton (PageRank + entry point detection). Supports `--tokens N`, `--focus "pattern"`, `--focus-changed`, `--json`, `--verbose`.
- `callers`/`callees`/`entry-points`/`file-graph`/`inspect`/`path`: graph query commands matching MCP tools.
- `query "term1|term2" . --exclude-test`: multi-term OR search with test file filtering.
- `diff . --markdown`: formats impact analysis report as color-coded Markdown.
- `check . --markdown`: executes safety threshold validation and prints Markdown PASS/FAIL status.
- `summary .`: auto-generates structured Pull Request descriptions based on graph diff analysis.
- `agent review`/`agent onboard`/`agent guard`: built-in autonomous workflows (PR risk review, contributor onboarding guide, architecture violation check with `--max-blast-radius`).
- `audit "sink"`: traces call paths to sensitive sinks (e.g. `db_query`, `exec`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Anandb71/arbor](https://github.com/Anandb71/arbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
