---
trigger: always_on
description: provides auto-fix
---

# LLM Agent Instructions for Panache

## Repository Overview

Panache is a formatter, linter, and LSP for Quarto (`.qmd`), Pandoc, and
Markdown files written in Rust. It understands Quarto/Pandoc-specific syntax
that other formatters struggle with (fenced divs, tables, math formatting).

**Syntax Reference**: `assets/pandoc-spec.md` contains comprehensive Pandoc
syntax specification with individual spec files in `assets/pandoc-spec/`. This
is the definitive reference for parser implementation.

### Key Facts

- **Language**: Rust 2024 edition, stable toolchain
- **Architecture**: Binary crate + WASM workspace for web playground
- **Status**: Early development
- **Tests**: 1,200+ total (846+ unit + 169 integration + others),
- **Parser**: Integrated inline parsing (single-pass, Pandoc-style) - only mode

### Principles

- Test-driven development: if you find a bug, write a test that reproduces it
  before fixing. If you want to add a new feature, write a test first.
- Pandoc parser is the gold standard - if in doubt, see how Pandoc handles it.
- Parsing failures take priority over formatting issues - the parser must be
  robust and lossless.

## Essential Commands

**Development workflow** (always run before making changes):

```bash
cargo check --workspace
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo fmt -- --check
```

**CLI testing:**

```bash
# Format in place
cargo run -- format document.qmd

# Format from file to stdout
cargo run -- format < document.qmd

# Format from stdin to stdout
cat document.qmd | cargo run -- format 

# Run parser+formatter debug checks (preferred over --verify flags)
cargo run -- debug format --checks all document.qmd

# Only run losslessness check
cargo run -- debug format --checks losslessness document.qmd

# Parse (show CST for debugging)
printf "# Test" | cargo run -- parse

# Targeted formatter golden case
cargo test --test golden_cases <case_name>

# Targeted parser CST/losslessness case
cargo test -p panache-parser --test golden_parser_cases <case_name>

# Lint
cargo run -- lint document.qmd
cargo run -- lint --fix document.qmd  # Apply auto-fixes

# LSP (for editor integration)
cargo run -- lsp
```

## Debugging with logging

```bash
# Debug parsing decisions (requires debug build via cargo run)
RUST_LOG=debug cargo run -- format document.qmd
RUST_LOG=trace cargo run -- parse document.qmd

# Module-specific
RUST_LOG=panache::parser::blocks=debug cargo run -- format document.qmd
RUST_LOG=panache::parser::inlines=debug cargo run -- format document.qmd

# Release builds: INFO logs only (DEBUG/TRACE compiled out)
RUST_LOG=info ./target/release/panache format document.qmd
```

**Shell command debugging tips:**

- Sync commands (default) return output directly - don't use `read_bash` on
  completed commands
- Only use `read_bash` if command is still running after `initial_wait`
- Use `mode="async"` for interactive sessions (REPL, debuggers)

## Core Architecture

### CST vs AST

**CST (Concrete Syntax Tree)**:

- Built with `rowan` crate - must preserve **every byte**, including whitespace
  and markers
- Essential for lossless parsing and LSP features
- Example: `ATX_HEADING_MARKER@0..1 "#"`, `WHITESPACE@1..2 " "`

**AST (Abstract Syntax Tree)**:

- Typed wrappers (`Heading`, `Link`, `Table`) hide syntactic details
- Pattern borrowed from rust-analyzer
- Example: `Heading::cast(node).level()` returns `1` without exposing `#`
  markers
- Located in
  `crates/panache-parser/src/syntax/{headings,links,tables,references}.rs`

### Single-Pass Parsing Architecture

**Parser** (`crates/panache-parser/src/parser/`):

1. **Parser** (`crates/panache-parser/src/parser/core.rs`):
   - Main parsing implementation in `Parser` struct
   - Parses block structures (headings, code blocks, paragraphs, tables, lists,
     etc.) in a single forward pass
   - Emits inline structure during block parsing (single-pass, Pandoc-style)
   - Each block type isolated in `blocks/` directory
   - Config-aware (respects flavor and extension flags)

2. **Inline Parsing** (integrated, not a separate pass):
   - Inline elements parsed during block parsing via `inlines/core.rs`
   - Delimiter-based with proper precedence (CommonMark spec)
   - Recursive for nested elements (e.g., emphasis in links)
   - Available in `inlines/` directory for specific constructs

3. **Block Dispatcher**
   (`crates/panache-parser/src/parser/block_dispatcher.rs`):
   - Centralized block detection via `detect_prepared()` and emission via
     `parse_prepared()` (legacy `can_parse/parse` removed)
   - Carries prepared payloads (e.g., blockquote markers, list metadata) to
     avoid re-parsing during emission
   - Parser core still owns continuation rules, blank-line handling, and list
     item buffering

**Key invariant**: Parser preserves ALL input bytes in CST, including structural
markers. Formatter applies formatting rules, not the parser.

### Formatter Architecture

Formatter core now lives in `crates/panache-formatter/`:

- `crates/panache-formatter/src/formatter/`: Core formatting implementation
  (paragraphs, inline, headings, lists, tables, etc.)
- `crates/panache-formatter/src/formatter.rs`: Core orchestration and YAML
  frontmatter formatting integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolars/panache](https://github.com/jolars/panache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
