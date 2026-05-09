---
trigger: always_on
description: Universal guide for AI coding agents (Claude Code, Codex, Gemini, Cursor, etc.) working on the Infiniloom project.
---

# AGENTS.md

Universal guide for AI coding agents (Claude Code, Codex, Gemini, Cursor, etc.) working on the Infiniloom project.

## Project Overview

**Infiniloom** is a high-performance repository context generator for Large Language Models, built in pure Rust. It transforms codebases into optimized formats for Claude, GPT-4o/GPT-5, Gemini, and other LLMs.

Key capabilities:

- AST-based symbol extraction using Tree-sitter (23 languages)
- PageRank-based symbol importance ranking
- Model-specific output formats (XML for Claude, Markdown for GPT, YAML for Gemini)
- Automatic secret detection and redaction
- Accurate token counting via tiktoken-rs (27 models across 9 providers)
- Native language bindings (Python via PyO3, Node.js via NAPI-RS)

## Build, Test, and Lint

All commands assume the working directory is the repo root.

```bash
# Build
cargo build --release                                    # Release binary -> ./target/release/infiniloom
cargo build --workspace                                  # Debug build (all crates)

# Test
cargo test --workspace                                   # All tests
cargo test --workspace -- --nocapture                    # With stdout
cargo test -p infiniloom-engine                          # Engine crate only
cargo test proptest                                      # Property-based tests

# Lint (strict -- CI enforces this)
cargo clippy --workspace --all-targets --all-features    # Must pass with zero warnings
cargo fmt --all -- --check                               # Formatting check

# Format
cargo fmt --all                                          # Auto-format all code

# Coverage
cargo llvm-cov --workspace --all-features --html --output-dir target/coverage
```

### Makefile Shortcuts

```bash
make build          # Debug build
make build-release  # Release build
make test           # All tests (--all-features)
make lint           # Strict clippy
make fmt            # Format code
make fmt-check      # Check formatting
make ci             # Full CI pipeline: fmt-check + lint + test + coverage
make pre-commit     # Quick checks: fmt-check + check + lint
```

### CI Expectations

The GitHub Actions pipeline (`.github/workflows/ci.yml`) runs on every PR to `main`:

1. `cargo fmt --check` -- formatting must be clean
2. `cargo clippy` -- zero warnings (correctness and perf are deny-level)
3. `cargo test --workspace` -- all tests pass on Ubuntu and macOS
4. `cargo audit` -- no known vulnerabilities
5. Python and Node.js binding builds
6. Code coverage uploaded to Codecov

**Before opening a PR, always run `make pre-commit` locally.**

## Architecture

### Workspace Layout

```
infiniloom/
├── cli/                     # CLI binary (clap-based)
│   └── src/
│       ├── main.rs          # Entry point, arg parsing
│       ├── config.rs        # Config loading
│       ├── scanner.rs       # Parallel file scanning (thread-local Tree-sitter parsers)
│       └── commands/        # One file per subcommand (pack, scan, map, diff, index, etc.)
├── engine/                  # Core library (infiniloom-engine)
│   └── src/
│       ├── lib.rs           # Public API
│       ├── types.rs         # Repository, RepoFile, Symbol, SymbolKind, CompressionLevel
│       ├── newtypes.rs      # Type-safe wrappers: SymbolId, FileId, LineNumber
│       ├── error.rs         # Error types
│       ├── parser/          # Tree-sitter AST parsing (23 languages)
│       ├── tokenizer/       # Multi-model token counting (27 models)
│       ├── repomap/         # PageRank symbol ranking
│       ├── output/          # Format generators (XML, Markdown, TOON/YAML)
│       ├── chunking/        # Semantic code chunking
│       ├── embedding/       # Vector DB chunk generation (BLAKE3 content-addressable)
│       ├── document/        # Document ingestion (MD, HTML, CSV, DOCX; feature-gated)
│       ├── index/           # Symbol index, dependency graph, call graph queries
│       ├── security.rs      # Secret detection and redaction
│       ├── config.rs        # YAML/TOML/JSON config loading
│       ├── git.rs           # Git CLI wrapper (std::process::Command, not libgit2)
│       ├── ranking.rs       # File importance scoring
│       ├── budget.rs        # Token budget management
│       └── incremental.rs   # File-level caching with change detection
├── bindings/
│   ├── common/              # Shared bindings utilities
│   ├── python/              # PyO3 + Maturin
│   └── node/                # NAPI-RS
└── packages/
    └── infiniloom/          # npm CLI wrapper
```

### Data Flow

1. **Scan** -- walk directory (respecting `.gitignore`), detect languages
2. **Parse** -- Tree-sitter AST extraction for symbols (parallel, thread-local parsers)
3. **Rank** -- PageRank importance scoring across symbol graph
4. **Format** -- model-specific output (XML/Markdown/YAML)
5. **Security** -- secret detection/redaction before output

### Feature Flags (engine/Cargo.toml)

| Flag | Dependencies | Purpose |
|------|-------------|---------|
| `document` (default) | zip, quick-xml | Document ingestion (MD, HTML, CSV, DOCX) |
| `document-xlsx` | calamine | XLSX spreadsheet support |
| `watch` | notify | File watching for `pack --watch` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Topos-Labs/infiniloom](https://github.com/Topos-Labs/infiniloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
