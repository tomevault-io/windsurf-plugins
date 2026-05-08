---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

﻿# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**delphi-lookup** is a high-performance Pascal source code search system designed for AI coding agents. It provides fast identifier lookup (~75ms end-to-end) across large Pascal codebases using COLLATE NOCASE indexes with short-circuit optimization.

**Features:**
- Fast identifier lookup: ~75ms end-to-end (~12ms search + ~65ms exe overhead) with short-circuit
- Full search: exact name + fuzzy matching + FTS5 full-text for non-identifier queries
- Framework-aware: VCL/FMX/RTL classification and filtering
- Documentation indexing: Official Delphi CHM help files
- Query caching: ~75ms end-to-end (~10ms search)
- Incremental indexing: Only processes new/modified files

> **Note on Vector/Semantic Search:** The codebase includes optional vector embeddings (via Ollama), but **FTS5-only is recommended** for AI agent workflows. Agents iterate fast and can achieve similar quality with multiple specific searches while being 17x faster. See "Vector Search Status" section for benchmark results.

## Tools

### delphi-indexer.exe
Builds searchable database from Pascal source folders and CHM documentation.

```bash
# Index source folders
delphi-indexer.exe "W:\YourProject" --category user
delphi-indexer.exe "C:\...\Studio\23.0\source\vcl" --category stdlib --framework VCL

# Index CHM documentation
delphi-indexer.exe --index-chm "C:\...\Help\Doc\vcl.chm" --delphi-version 12.0

# List indexed folders
delphi-indexer.exe --list-folders
```

**Key parameters:**
- `--category` - Source classification: `user`, `stdlib`, `third_party`, `official_help`
- `--framework` - Force framework tag: `VCL`, `FMX`, `RTL` (skips auto-detection)
- `--type` - Content type: `code`, `help`, `markdown`
- `--force` - Full reindex (ignore timestamps/hashes)
- `--index-chm` - Index CHM documentation file
- `--delphi-version` - Version tag for CHM (default: 12.0)

### delphi-lookup.exe
Fast hybrid search with caching and framework filtering.

```bash
# Basic searches
delphi-lookup.exe "TStringList" -n 5
delphi-lookup.exe "JSON serialization" -n 3

# Framework filtering
delphi-lookup.exe "TButton" --framework VCL -n 5
delphi-lookup.exe "TForm" --framework FMX -n 3

# Category filtering
delphi-lookup.exe "ShowMessage" --category stdlib -n 5
delphi-lookup.exe "TMyClass" --prefer user -n 10

# Symbol type filtering
delphi-lookup.exe "MAX_BUFFER" --symbol const
delphi-lookup.exe "ValidateInput" --symbol function

# Advanced
delphi-lookup.exe "validation logic" --use-reranker --candidates 100
```

**Key parameters:**
- `-n` - Number of results (default: 5)
- `--framework` - Filter by framework: `VCL`, `FMX`, `RTL`
- `--category` - Filter by category: `user`, `stdlib`, `third_party`, `official_help`
- `--prefer` - Boost category in results (doesn't exclude others)
- `--symbol` - Filter by type: `class`, `function`, `const`, `var`, etc.
- `--use-reranker` - Enable two-stage reranking (improves precision ~75% → ~95%, +100ms)
- `--candidates` - Candidate count for reranking (default: 50)

### CheckFTS5.exe
Diagnostic tool to verify FTS5 support in sqlite3.dll.

```bash
./CheckFTS5.exe
```

## Configuration

delphi-lookup supports configuration via JSON file, environment variables, or command-line parameters (in order of precedence: CLI > env > config file).

### Configuration File

Copy `delphi-lookup.example.json` to `delphi-lookup.json`:

```json
{
  "database": "delphi_symbols.db",
  "buffer_size": 500,
  "category": "user",
  "num_results": 5
}
```

> **Note:** FTS5-only mode is the default and recommended configuration. Embedding parameters exist but are not recommended (see "Vector Search Status" section).

### Configuration Parameters

| Parameter | Default | Used By | Description |
|-----------|---------|---------|-------------|
| `database` | `delphi_symbols.db` | Both | SQLite database file |
| `buffer_size` | `500` | Indexer | Batch size for processing |
| `category` | `user` | Indexer | Default source category |
| `num_results` | `5` | Search | Number of results to return |

### Experimental Parameters (Not Recommended)

These parameters enable vector embeddings and reranking. **Not recommended for AI agent workflows** - see "Vector Search Status" section for rationale.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `embedding_url` | `""` | Embedding service URL (empty = disabled) |
| `semantic_search` | `false` | Enable vector similarity search |
| `use_reranker` | `false` | Enable two-stage reranking |

### Special Behaviors

- **No config file**: Use `--no-config` to skip loading `delphi-lookup.json`

## Framework Detection System

delphi-lookup includes intelligent, multi-tier framework detection to accurately categorize mixed RTL/VCL/FMX code.

### Package Intelligence

```bash
# Scan .dpk packages to populate package database
delphi-indexer.exe --scan-packages "C:\Projects\Packages" --category user

# Scan Delphi standard library packages
delphi-indexer.exe --scan-packages "C:\...\Studio\23.0\lib" --category stdlib

# List all scanned packages
delphi-indexer.exe --list-packages
```

**Framework detection from packages:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JavierusTk/delphi-lookup](https://github.com/JavierusTk/delphi-lookup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
