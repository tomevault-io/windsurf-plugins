---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
# Setup git hooks (run once after clone)
lefthook install

# Format code
cargo fmt

# Run linter
cargo clippy --all-targets --all-features -- -D warnings

# Run all tests
cargo test

# Run tests with coverage
cargo llvm-cov --all-features --ignore-filename-regex 'main\.rs$'

# Build release binary
cargo build --release

# Run a single test
cargo test test_complexity_simple

# Run tests for a specific module
cargo test analyzers::complexity
```

## Architecture

Omen is a multi-language code analysis CLI built in Rust. It uses tree-sitter for parsing source code across 13 languages.

### Module Structure

```
src/
  cli/           - CLI entry point using clap
  config/        - Configuration loading (TOML)
  core/          - Core types and traits
  analyzers/     - Analysis implementations
    complexity/  - Cyclomatic and cognitive complexity
    satd/        - Self-admitted technical debt
    deadcode/    - Unused code detection
    churn/       - Git history file churn
    clones/      - Code clone detection (MinHash+LSH)
    defect/      - Defect probability (PMAT)
    changes/     - JIT commit-level risk
    diff/        - PR/branch diff analysis
    tdg/         - Technical Debt Gradient
    graph/       - Dependency graph (Mermaid)
    hotspot/     - High churn + complexity
    temporal/    - Temporal coupling
    ownership/   - Code ownership and bus factor
    cohesion/    - CK metrics (WMC, CBO, RFC, LCOM4, DIT, NOC)
    repomap/     - PageRank-ranked symbols
    smells/      - Architectural smells (Tarjan SCC)
    flags/       - Feature flag detection
    mutation/    - Mutation testing (21 operators, parallel execution)
  git/           - Git operations (log, blame, diff)
  parser/        - Tree-sitter wrapper
  mcp/           - MCP server for LLM integration
  output/        - Output formatting (JSON/Markdown/text)
  score/         - Repository health scoring
```

### Key Patterns

**Analyzer pattern**: Each analyzer module follows the same structure:
1. Public `analyze()` function taking path and options
2. Returns a result struct with analysis data
3. Implements `Serialize` for JSON output
4. Uses rayon for parallel file processing

**Multi-language parsing**: `parser/mod.rs` contains `Language` enum and `Parser` struct. Add new language support by:
1. Adding variant to `Language` enum
2. Implementing tree-sitter grammar in `parser()`
3. Adding node types in extraction functions

**Concurrent file processing**: Uses rayon's parallel iterators:
```rust
files.par_iter()
    .filter_map(|path| analyze_file(path).ok())
    .collect()
```

**Configuration**: Config loaded from `omen.toml` or `.omen/omen.toml`. See `omen.example.toml` for all options.

**MCP server**: JSON-RPC server in `mcp/` module exposing all analyzers as tools for LLM integration. Tool names are bare analyzer names (e.g., `complexity`, `satd`, `temporal`) -- no prefix.

**`--since` flag**: Commands that accept `--since` (e.g., `report generate`, `score trend`) default to `"all"` (full repo history). The value `"all"` is handled by `is_since_all()` in `src/git/log.rs`, which causes `parse_since_to_days()` to return `None` (no time limit). Duration values like `3m`, `6m`, `1y` still work.

### CLI Commands

Top-level commands (flat structure):
- `complexity` - Cyclomatic and cognitive complexity
- `satd` - Self-admitted technical debt
- `deadcode` - Unused code detection
- `churn` - Git history file churn
- `clones` - Code clone detection
- `defect` - Defect probability prediction
- `changes` - Commit-level change risk (JIT)
- `diff` - Branch diff risk analysis
- `tdg` - Technical Debt Gradient
- `graph` - Dependency graph
- `hotspot` - High churn + complexity files
- `temporal` - Temporal coupling
- `ownership` - Code ownership and bus factor
- `cohesion` - CK object-oriented metrics
- `repomap` - PageRank-ranked symbol map
- `smells` - Architectural smell detection
- `flags` - Feature flag detection
- `mutation` - Mutation testing (21 operators across 5 languages)
- `score` - Repository health score
- `all` - Run all analyzers
- `context` - Deep context for LLMs
- `report` - HTML health reports
- `mcp` - Start MCP server

**Global flags**: `-p/--path`, `-f/--format`, `-c/--config`, `-v/--verbose`, `-j/--jobs`, `--no-cache`, `--ref`, `--shallow`

### Report System

`omen report generate` runs all analyzers in parallel, then invokes LLM analyst agents to produce insight narratives, and renders an HTML report via `src/report/render.rs`.

Key files:
- `src/report/render.rs` -- loads JSON data files + optional insight JSON files, renders HTML
- `src/report/types.rs` -- all Rust types for report data and insights (must match agent output schemas)
- `src/report/template.html` -- Handlebars HTML template
- `plugins/reporting/commands/generate-report.md` -- orchestration command (runs analyzers, spawns agents)
- `plugins/reporting/agents/` -- 12 analyst agents that produce `{section_insight: string}` JSON


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panbanda/omen](https://github.com/panbanda/omen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
