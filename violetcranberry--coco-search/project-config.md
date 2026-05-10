---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, OpenCode, and others) when working with code in this repository. It is the single source of truth — `AGENTS.md` is a symlink to this file.
---

# Project Instructions

This file provides guidance to AI coding agents (Claude Code, OpenCode, and others) when working with code in this repository. It is the single source of truth — `AGENTS.md` is a symlink to this file.

## Project Overview

CocoSearch is a local-first hybrid semantic code search tool powered by CocoIndex and Tree-sitter. It indexes codebases into PostgreSQL with pgvector embeddings and provides search through CLI, MCP server, or interactive REPL. Local by default with Ollama; optional remote embedding providers (OpenAI, OpenRouter) available for teams that prefer managed infrastructure. Requires Python >=3.11.

## Tool Routing (MANDATORY)

When CocoSearch MCP tools are available, ALWAYS use them instead of Grep, Glob, or Task/Explore agents for code search and exploration. These rules are mandatory, not advisory. Violations degrade search quality and create unnecessary permission prompts.

| Task | Use this | NOT this |
|------|----------|----------|
| Code search / "how does X work?" | `search_code` | Grep, Glob, Task (Explore) |
| Symbol lookup / "find function Y" | `search_code` with `symbol_name`/`symbol_type` | Grep for def/class patterns |
| Dependency tracing / "what imports X?" | `get_file_dependencies` / `get_file_impact` | Grep for import statements |
| Batch dependency analysis (multiple files) | `get_batch_dependencies` / `get_batch_impact` | Per-file `get_file_dependencies` calls |
| Search debugging / "why no results?" | `analyze_query` | Manual pipeline investigation |

Fall back to Grep/Glob ONLY for:
- Exact literal string matches (e.g., a specific error message or config value)
- File path pattern matching (e.g., "find all `*.test.ts` files")
- Editing operations that need line numbers from a known file

## Development Setup

```bash
# Prerequisites: Docker, uv (Python package manager)
# One-command setup (starts infra, pulls model, installs deps, indexes codebase):
./dev-setup.sh

# Or manually:
docker compose --profile ollama up -d    # PostgreSQL 17 + Ollama
uv sync                                 # Install dependencies
uv run cocosearch index .               # Index the codebase
```

**Infrastructure:** PostgreSQL 17 (pgvector) on port 5432, Ollama on port 11434. Defaults require no `.env` file.

## Commands

```bash
# Run all unit tests (default, mocked, no infra needed). Takes a long time.
uv run pytest

# Run a single test file
uv run pytest tests/unit/search/test_cache.py -v

# Run a single test by name
uv run pytest -k "test_rrf_double_match_ranks_higher" -v

# Run handler tests
uv run pytest tests/unit/handlers/ -v

# Lint and format
uv run ruff check src/ tests/
uv run ruff check --fix src/ tests/     # Auto-fix lint issues
uv run ruff format src/ tests/          # Format code

# CLI usage
uv run cocosearch index .
uv run cocosearch search "query"
uv run cocosearch search -i          # Interactive REPL
uv run cocosearch search -i --indexes "repo_a,repo_b"  # Cross-index interactive REPL
uv run cocosearch search --indexes "repo_a,repo_b" "query"  # Cross-index search
uv run cocosearch analyze "query"    # Pipeline analysis with diagnostics
uv run cocosearch analyze "query" --json  # JSON pipeline analysis
uv run cocosearch analyze --indexes "repo_a,repo_b" "query"  # Cross-index analysis
uv run cocosearch stats
uv run cocosearch list
uv run cocosearch clear <index>
uv run cocosearch clear idx1 idx2    # Delete multiple indexes
uv run cocosearch clear --all        # Delete all indexes
uv run cocosearch languages              # List supported languages
uv run cocosearch grammars               # List supported grammars
uv run cocosearch init                   # Initialize cocosearch.yaml + optional CLAUDE.md/AGENTS.md
uv run cocosearch init --no-claude-md    # Initialize without CLAUDE.md prompt
uv run cocosearch init --no-agents-md    # Initialize without AGENTS.md prompt
uv run cocosearch init --no-opencode-mcp # Initialize without OpenCode MCP registration prompt
uv run cocosearch init --no-opencode-skills # Initialize without OpenCode skills installation prompt
uv run cocosearch init --no-claude-mcp   # Initialize without Claude Code plugin prompt
uv run cocosearch init --no-claude-settings # Initialize without Claude Code permissions prompt
uv run cocosearch config show
uv run cocosearch config path
uv run cocosearch config check
uv run cocosearch dashboard              # Terminal dashboard

# Dependency graph (incremental by default, use --fresh for full re-extraction)
uv run cocosearch index . --deps          # Index + extract dependencies
uv run cocosearch deps extract .          # Extract dependencies (incremental)
uv run cocosearch deps extract . --fresh  # Force full re-extraction
uv run cocosearch deps show <file>        # Show dependencies for a file
uv run cocosearch deps tree <file>        # Forward dependency tree (transitive)
uv run cocosearch deps impact <file>      # Reverse impact tree (what depends on this)
uv run cocosearch deps stats              # Show dependency graph statistics

# MCP server
uv run cocosearch mcp --project-from-cwd
```

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VioletCranberry/coco-search](https://github.com/VioletCranberry/coco-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
