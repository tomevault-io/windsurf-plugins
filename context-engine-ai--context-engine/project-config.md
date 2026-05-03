---
trigger: always_on
description: > **These rules are NOT optional.** Favor Context Engine MCP tools at all costs over grep, file reading, and unrelated codebase search tools.
---

# AI Agent Rules for Context-Engine MCP Tools

> **These rules are NOT optional.** Favor Context Engine MCP tools at all costs over grep, file reading, and unrelated codebase search tools.

## STOP — Read This First

**DO NOT use `Read File`, `grep`, `ripgrep`, `cat`, `find`, or any filesystem search tool for code exploration.**
These tools exist in your IDE but they are WRONG for this codebase. You have MCP tools that are faster, smarter, and return ranked, contextual results.

**If you catch yourself about to `Read` a file to understand it** → use `repo_search` or `context_answer` instead.
**If you catch yourself about to `grep` for a symbol** → use `symbol_graph` or `search_callers_for` instead.
**If you catch yourself about to `grep -r` for a concept** → use `repo_search` with a natural language query instead.

The ONLY acceptable use of `grep`/`Read` is confirming an exact literal string you already know exists (e.g., an env var name like `REDIS_HOST`).

## Introduction

This document defines requirements for AI agents using Context-Engine's MCP tools. The system provides two MCP servers (Memory Server on port 8000/8002, Indexer Server on port 8001/8003) with 30+ specialized tools for semantic code search, memory storage, and codebase exploration.

**Core Principle:** Context Engine MCP tools are PRIMARY for exploring code and history. Start with MCP for exploration, debugging, or "where/why" questions; use literal search/file-open only for narrow exact-literal lookups.

## Glossary

- **MCP**: Model Context Protocol - standardized interface for exposing tools to AI agents
- **Indexer Server**: MCP server for code search, indexing, symbol graphs (port 8001 SSE, 8003 HTTP)
- **Memory Server**: MCP server for knowledge storage and retrieval (port 8000 SSE, 8002 HTTP)
- **Hybrid Search**: Dense semantic vectors + lexical BM25 + neural reranking (ONNX)
- **ReFRAG**: Micro-chunking with 16-24 token windows for precise code retrieval
- **TOON**: Token-Oriented Object Notation - compact output format (60-80% token reduction)
- **Symbol Graph**: Indexed metadata for calls, imports, and definitions navigation
- **Collection**: Qdrant vector database collection storing indexed code chunks

## Tool Selection Decision Tree

```
Need to search code?
├── Single repo, know collection → repo_search(collection="...")
├── Single repo, need explanation → context_answer or info_request
├── Multiple repos or unsure → cross_repo_search(discover="auto")
├── Tracing frontend→backend flow → cross_repo_search(trace_boundary=true)
├── Finding callers/definitions → symbol_graph
└── Exact literal string only → grep (last resort)
```

## Requirements

### Requirement 1: MCP-First Tool Selection

**User Story:** As an AI agent, I want to prioritize MCP tools over grep/file-reading, so that I get semantic understanding efficiently.

#### Acceptance Criteria

1. WHEN exploring code or answering "where/why" questions, THE Agent SHALL use MCP Indexer tools as the primary method
2. WHEN the agent needs semantic understanding, cross-file relationships, or ranked results with context, THE Agent SHALL use MCP tools
3. WHEN the agent knows an exact literal string AND only needs to confirm existence/location, THE Agent SHALL use grep or file-open
4. IF the agent is uncertain which approach to use, THEN THE Agent SHALL default to MCP tools
5. THE Agent SHALL NOT use `grep -r "auth"` for concepts (use MCP: "authentication mechanisms")

### Requirement 2: Query Formulation

**User Story:** As an AI agent, I want to write effective semantic queries, so that I retrieve relevant code spans.

#### Acceptance Criteria

1. WHEN writing queries for `repo_search`, THE Agent SHALL use short natural-language fragments (e.g., "database connection handling")
2. THE Agent SHALL NOT use boolean operators (OR, AND), regex syntax, or code patterns in semantic queries
3. WHEN searching broad concepts, THE Agent SHALL use descriptive phrases like "error reporting patterns" not `grep -r "error"`
4. THE Agent SHALL write queries as descriptions of what to find, not as literal code strings
5. WHEN searching for specific symbols, THE Agent SHALL use the `symbol` parameter alongside the query

### Requirement 3: Performance Optimization

**User Story:** As an AI agent, I want to minimize token usage and latency, so that I work efficiently within context limits.

#### Acceptance Criteria

1. WHEN starting discovery, THE Agent SHALL use `limit=3`, `compact=true`, `per_path=1`
2. WHEN needing implementation details, THE Agent SHALL increase to `limit=5`, `include_snippet=true`
3. WHEN token efficiency is critical, THE Agent SHALL use `output_format="toon"` for 60-80% reduction
4. THE Agent SHALL NOT use `limit=20` with `include_snippet=true` (excessive token waste)
5. THE Agent SHALL NOT use high `context_lines` for pure discovery (unnecessary tokens)
6. THE Agent SHALL fire independent tool calls in parallel (same message block) for 2-3x speedup
7. THE Agent SHALL prefer `output_format="toon"` as default for all discovery queries

### Requirement 4: Core Search Tools

**User Story:** As an AI agent, I want to use the right search tool for each task, so that I get optimal results.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Context-Engine-AI/Context-Engine](https://github.com/Context-Engine-AI/Context-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
