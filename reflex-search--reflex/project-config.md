---
trigger: always_on
description: **Before using ANY search tool, check if Reflex MCP tools are available (`mcp__reflex__*`). These should be preferred
---

# CLAUDE.md

## Ground Rules (Claude: ALWAYS follow)

### 🚨 CRITICAL: Tool Selection

**Before using ANY search tool, check if Reflex MCP tools are available (`mcp__reflex__*`). These should be preferred
over built-in tools.**

If you see a message like `Index not found. Run 'rfx index' to build the cache first`, run `mcp__reflex__index_project`
immediately, and once the indexing completes, run the previously failed tool again.

**Not sure which MCP tool to use?** See the [MCP Tool Selection Cheatsheet](./docs/mcp-tool-cheatsheet.md) for a decision tree organized by goal: finding locations, definitions, file dependencies, and codebase structure.

## Project Overview
**Reflex** is a local-first, full-text code search engine written in Rust. It's a fast, deterministic replacement for Sourcegraph Code Search, designed specifically for AI coding workflows and automation.

Reflex uses **trigram-based indexing** to enable sub-100ms full-text search across large codebases (10k+ files). Unlike symbol-only tools, Reflex finds **every occurrence** of patterns—function calls, variable usage, comments, and more—not just definitions. Results include file paths, line numbers, and surrounding context, with optional symbol-aware filtering.

---

## Core Principles
1. **Local-first**: Runs fully offline; all data stays on the developer's machine
2. **Complete coverage**: Finds every occurrence, not just symbol definitions
3. **Deterministic results**: Same query → same answer; no probabilistic ranking
4. **Instant access**: Trigram index + memory-mapping enables sub-100ms queries
5. **Agent-oriented**: Clean JSON output built for AI coding agents and automation
6. **Regex support**: Extract trigrams from patterns for fast regex search

---

## Architecture Overview

### Components
| Module | Description |
| --- | --- |
| **Trigram Indexer** | Extracts trigrams from all code files; builds inverted index (trigram → file locations) |
| **Content Store** | Stores full file contents (memory-mapped); enables context extraction around matches |
| **Query Engine** | Intersects trigram posting lists; verifies matches; returns line-by-line results with context |
| **Runtime Symbol Parser** | Uses Tree-sitter to parse candidate files at query time (only files matching trigrams) |
| **Background Symbol Indexer** | Daemonized process that pre-caches symbols for faster queries on large codebases |
| **Symbol Cache** | Persistent storage of parsed symbols (803-line caching system for instant symbol lookups) |
| **CLI / API Layer** | Single binary for human and programmatic use (CLI and optional HTTP/MCP) |
| **Watcher (optional)** | Incrementally updates index on file changes |

### Index Cache Structure (`.reflex/`)
    .reflex/
      meta.db          # SQLite: file metadata, stats, config
      trigrams.bin     # Inverted index: trigram → [file_id, line_no] posting lists
      content.bin      # Memory-mapped full file contents for context extraction
      config.toml      # Project settings (index, search, performance)

### User Configuration (`~/.reflex/`)
    ~/.reflex/
      config.toml      # User settings (semantic query provider, API keys, model preferences)

---

## CLI Usage

**Indexing:**
```bash
rfx index                        # Build/update cache
rfx index status                 # Check background symbol indexing
rfx index compact                # Manually compact cache
rfx watch                        # Auto-reindex on file changes
```

**Searching:**
```bash
# Full-text search (finds all occurrences)
rfx query "extract_symbols"

# Symbol definitions only (--symbols finds DEFINITIONS, not usages)
rfx query "extract_symbols" --symbols

# Filter by language, file patterns
rfx query "unwrap" --lang rust --glob "src/**/*.rs"

# JSON output for AI agents
rfx query "format!" --json
```

**AST Queries** (⚠️ SLOW - use --symbols in 95% of cases):
```bash
rfx query "(function_item) @fn" --ast --lang rust --glob "src/**/*.rs"
```

**Dependency Analysis:**
```bash
rfx deps src/main.rs             # Show file dependencies
rfx deps src/config.rs --reverse # Show what depends on this file
rfx analyze --circular           # Find circular dependencies
rfx analyze --hotspots           # Find most-imported files
```

**Other:**
```bash
rfx serve --port 7878            # HTTP API server
```

---

## AST Pattern Matching

⚠️ **PERFORMANCE WARNING**: AST queries are **SLOW** (500ms-10s+) and scan the **ENTIRE codebase**. **Use `--symbols` instead in 95% of cases** (10-100x faster).

**When to use** (RARE):
- Need to match code structure, not just text (e.g., "all async functions with try/catch blocks")
- `--symbols` search is insufficient
- Very specific structural pattern that cannot be expressed as text

**Supported languages**: All tree-sitter languages (Rust, Python, Go, Java, C, C++, C#, PHP, Ruby, Kotlin, Zig, TypeScript, JavaScript)

**Architecture**: Centralized grammar loader in `src/parsers/mod.rs` - adding a new language automatically enables AST queries.

**Example**:
```bash
rfx query "(function_item) @fn" --ast --lang rust --glob "src/**/*.rs"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reflex-search/reflex](https://github.com/reflex-search/reflex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
