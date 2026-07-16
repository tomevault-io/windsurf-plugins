---
trigger: always_on
description: This file defines the capabilities and constraints for AI agents working with this codebase.
---

# MCP Server Agent Configuration

This file defines the capabilities and constraints for AI agents working with this codebase.

## Current State

**System Complexity**: 5/5 (High — SQLite FTS5 + Qdrant vector index, 48 language plugins, rerankers, query-intent routing)
**MCP Status**: Use MCP indexed search when repository readiness is `ready`; STDIO is the primary surface
**Last Updated**: 2026-04-23
**Support matrix**: Customer-facing language/runtime support claims live in `docs/SUPPORT_MATRIX.md`.
**Dependency truth**: Use `uv sync --locked`; `pyproject.toml` and `uv.lock` are canonical.

> **Beta status**: Multi-repo support and the STDIO interface are in beta. STDIO is the primary surface for LLM tool calls; FastAPI is a secondary admin surface for diagnostics and manual operations. Expect API surface changes before stable release.
>
> **Public alpha repository model**: v3 supports many unrelated repositories on
> one machine, with one registered worktree per git common directory. Only the
> tracked/default branch is indexed automatically. Indexed results are
> authoritative only when readiness is `ready`; unavailable indexes return
> `index_unavailable` with `safe_fallback: "native_search"`.

### What's Actually Implemented
- ✅ STDIO transport (`search_code`, `symbol_lookup`, `summarize_sample`, `reindex` MCP tools)
- ✅ FastAPI admin surface with endpoints: `/symbol`, `/search`, `/status`, `/plugins`, `/reindex`
- ✅ Dispatcher with caching and auto-initialization
- ✅ Python plugin fully functional with Tree-sitter + Jedi
- ✅ JavaScript/TypeScript plugin fully functional with Tree-sitter
- ✅ C plugin fully functional with Tree-sitter
- ✅ SQLite persistence layer with FTS5 search
- ✅ File watcher auto-starts in `initialize_services()` after dispatcher is ready; stopped in `main()` finally block on exit (EnhancedDispatcher mode only)
- ✅ Error handling and logging framework
- ✅ Comprehensive testing framework (pytest with fixtures)
- ✅ CI/CD pipeline with GitHub Actions
- ✅ Docker support and build system

### What's Recently Implemented
- ✅ C++, HTML/CSS, and Dart plugins fully functional with Tree-sitter
- ✅ Advanced metrics collection with Prometheus
- ✅ Security layer with JWT authentication
- ✅ Comprehensive testing framework with parallel execution
- ✅ Docker and Kubernetes configurations (beta hardening in progress)
- ✅ Cache management and query optimization
- ✅ Real-world repository testing validation

## Agent Capabilities

### Code Understanding
- Parse and understand the intended architecture
- Navigate plugin structure (though most are stubs)
- Interpret C4 architecture diagrams
- Understand the gap between design and implementation

### Code Modification
- Add new language plugin stubs
- Extend API endpoint definitions
- Update architecture diagrams
- Implement missing functionality

### Testing & Validation
- Run basic test files (`test_python_plugin.py`, `test_tree_sitter.py`)
- Validate TreeSitter functionality
- Check architecture consistency
- Identify implementation gaps

## MCP SEARCH STRATEGY (CRITICAL)

### Use Indexed Search When Readiness Is Ready
The codebase can use a pre-built index across 48 languages. Before treating
indexed results as authoritative, check `mcp__code-index-mcp__get_status()` for
repository readiness `ready` or honor the query tool response. If `search_code`
or `symbol_lookup` returns `code: "index_unavailable"` with
`safe_fallback: "native_search"`, use native `rg`/file tools and follow the
readiness remediation, usually `reindex`.

### Tool Priority Order:
1. **mcp__code-index-mcp__symbol_lookup** - For finding definitions when readiness is `ready`
   - Use for: Classes, functions, methods, variables
   - Returns: Exact location, signature, documentation
   - Speed: <100ms
   - `result: "not_found"` from a ready index means no symbol match; `index_unavailable` means use `native_search`
   - Example: `mcp__code-index-mcp__symbol_lookup(symbol="PluginManager")`

2. **mcp__code-index-mcp__search_code** - For pattern/content search when readiness is `ready`
   - Use for: Code patterns, text search, semantic queries
   - Supports: Regex, semantic search with semantic=true
   - Speed: <500ms, returns ranked results with line numbers and `last_indexed` timestamp
   - `results: []` from a ready index means no code match; `index_unavailable` means use `native_search`
   - Example: `mcp__code-index-mcp__search_code(query="def.*process", limit=10)`
   - Semantic: `mcp__code-index-mcp__search_code(query="authentication flow", semantic=true)`

3. **Native tools (`rg`, file reads)** - Safe fallback for non-ready indexes
   - Use when MCP tools are unavailable or return `safe_fallback: "native_search"`
   - Use for reading specific files after indexed search identifies candidates
   - Use while `reindex` or other readiness remediation is pending

### Examples:
Check readiness first:
`mcp__code-index-mcp__get_status()`

When readiness is `ready`:
`mcp__code-index-mcp__search_code(query="class.*Plugin")`

When `index_unavailable` is returned:
use `rg` or file tools and follow the returned remediation.

### Performance Impact:
- Traditional grep through 312 files: ~45 seconds
- MCP indexed search: <0.5 seconds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Consiliency/Code-Index-MCP](https://github.com/Consiliency/Code-Index-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
