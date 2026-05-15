---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Skim** is a streaming code reader for AI agents built in Rust using tree-sitter. It transforms source code by stripping implementation details while preserving structure, signatures, and types - optimizing code for LLM context windows.

**Key Principle:** This is a **streaming reader** (like `cat` but smart), NOT a file compression tool. Output always goes to stdout for pipe workflows.

## Current Project State

✅ **PHASE 3 COMPLETE** (100% of original roadmap)

**What's Complete (Phases 1 & 2):**
- ✅ Full Rust project with comprehensive test suite (3,103 tests passing)
- ✅ 17 languages supported: TypeScript, JavaScript, Python, Rust, Go, Java, C, C++, C#, Ruby, SQL, Kotlin, Swift, Markdown, JSON, YAML, TOML
- ✅ 4 transformation modes: structure, signatures, types, full
- ✅ CLI with stdin/stdout streaming support
- ✅ Multi-file glob support with parallel processing
- ✅ Published to crates.io (`cargo install rskim`)
- ✅ Published to npm (`npm install -g rskim` / `npx rskim`)
- ✅ Cross-platform binaries (Linux, macOS x64/ARM, Windows)
- ✅ CI/CD pipeline with automated releases
- ✅ Comprehensive documentation (README, CLAUDE.md, API docs)
- ✅ Performance benchmarks verified (14.6ms for 3000-line files)

**Phase 3 Status:**
- ✅ Multi-file glob support (`rskim 'src/**/*.ts'`) - COMPLETE
- ✅ Parallel processing with rayon (`--jobs` flag for multi-file operations) - COMPLETE
- ✅ Performance benchmarks (verified: 14.6ms for 3000-line files) - COMPLETE
- ✅ Parser caching layer (`~/.cache/skim/` with mtime invalidation) - COMPLETE
  - Enabled by default, 40-50x speedup on cached reads
  - `--no-cache` flag to disable, `--clear-cache` to clear
  - Implemented at CLI layer, core library remains pure
- ✅ Token counting feature (`--show-stats`) - COMPLETE
  - Uses tiktoken (cl100k_base for GPT-3.5/GPT-4)
  - Works with single files, globs, and stdin
  - Output to stderr for clean piping

## Technology Stack

- **Language:** Rust (performance, zero-cost abstractions)
- **Parser:** tree-sitter (multi-language AST parsing)
- **CLI:** clap with derive API
- **Output:** Streaming to stdout via `BufWriter`
- **Distribution:** cargo-dist (cross-platform binaries + npm publishing)
- **Performance Target:** <50ms for 1000-line files

## Architecture

```
Parser Manager (language detection)
  ↓
┌─────────────────────────────────────┐
│     Strategy Pattern Dispatcher      │
│   Language::transform_source()      │
└─────────────┬───────────────────────┘
              │
    ┌─────────┴─────────┐
    ↓                   ↓
tree-sitter         serde-based
(TS/JS/Python/      (JSON, YAML, TOML)
 Rust/Go/Java/
 C/C++/MD)
    ↓                   ↓
Transformation Layer (modes: structure/signatures/types/full)
  ↓
Streaming Output (stdout, zero-copy when possible)

analytics/          ← Token savings persistence (SQLite)
  mod.rs            ← AnalyticsDb, AnalyticsStore trait, fire-and-forget recording
  schema.rs         ← Versioned migrations (v1: token_savings, v2: analytics_meta)
```

**ARCHITECTURE NOTE**: JSON, YAML, and TOML use serde-based parsers instead of tree-sitter because they are data formats, not code. The Strategy Pattern in `Language::transform_source()` routes each language to its appropriate parser, eliminating special-case conditionals.

**ANALYTICS NOTE**: The `analytics/` module persists token savings to `~/.cache/skim/analytics.db` (SQLite with WAL mode). Recording is fire-and-forget via background threads. The `AnalyticsStore` trait enables MockStore-based testing of the stats dashboard without a real database. `--clear-cache` does NOT touch `analytics.db`.

## Implementation Phases

### Phase 1 (Weeks 1-4): Proof of Concept
- Single language (TypeScript)
- Basic structure extraction (strip function bodies)
- CLI with mode flags
- Streaming stdout output

### Phase 2 (Weeks 5-8): Multi-Language
- 5 languages (TypeScript, Python, Rust, Go, Java)
- Language detection from file extensions
- Performance optimization (<50ms target)
- CI pipeline

### Phase 3 (Weeks 9-12): Production
- Caching layer (mtime-based)
- Multi-file/glob support
- Parallel processing (rayon)
- Binary releases (cargo-dist for crates.io + npm)

## Installation

### For End Users

```bash
# Via Homebrew (macOS/Linux)
brew install dean0x/tap/skim

# Via npm (recommended - try without installing)
npx rskim file.ts

# Via npm (global install for regular use)
npm install -g rskim

# Via cargo (recommended for Rust developers)
cargo install rskim

# Via binary download (GitHub releases)
curl -L https://github.com/dean0x/skim/releases/latest/download/rskim-x86_64-unknown-linux-gnu.tar.gz | tar xz
```

✅ **Package naming:** Successfully published as `rskim` on both npm and crates.io.

## Commands

**Local dev binary:** `target/release/skim` is on PATH for local testing. After every merge to main, run `cargo build --release` to keep it current.

### Build & Test
```bash
cargo build --release          # Production build
cargo test                     # Run test suite
cargo test --all-features      # Run all tests
cargo bench                    # Run benchmarks
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dean0x/skim](https://github.com/dean0x/skim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
