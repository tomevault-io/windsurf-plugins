---
trigger: always_on
description: A fast Rust codebase indexer for AI agents. Extracts structural maps (declarations, imports, tree) using tree-sitter and regex parsing across 27 languages.
---

# indxr

A fast Rust codebase indexer for AI agents. Extracts structural maps (declarations, imports, tree) using tree-sitter and regex parsing across 27 languages.

## Codebase Navigation — MUST USE indxr MCP tools

An MCP server called `indxr` is available. **Always use indxr tools before the Read tool.** Do NOT read full source files as a first step — use the MCP tools to explore, then read only what you need.

### Token savings reference

The MCP server defaults to **3 compound tools** (`find`, `summarize`, `read`). All 26 tools (3 compound + 23 granular) are available with `--all-tools`. With `--features wiki`, 9 additional wiki tools are available.

| Action | Approx tokens | When to use |
|--------|--------------|-------------|
| `find(query)` | ~100-400 | Find files/symbols by concept, name, callers, or signature pattern |
| `summarize(path)` | ~200-600 | Understand a file, batch of files, or symbol without reading source |
| `read(path, symbol?)` | ~50-300 | Read one function/struct. Supports `symbols` array and `collapse`. |
| `Read` (full file) | **500-10000+** | ONLY when editing or need exact formatting |

**Typical exploration: ~500 tokens vs ~3000+ for reading a full file (6x reduction).**

### Exploration workflow (follow this order)

The default 3 compound tools cover the most common exploration patterns:

1. `find(query)` — find files/symbols by concept, partial name, or type pattern. **Start here when you know what you're looking for but not where it is.**
   - Default mode (`relevant`): multi-signal relevance search across paths, names, signatures, and docs. Supports `kind` filter.
   - `mode: "symbol"`: find declarations by name (case-insensitive substring).
   - `mode: "callers"`: find who references a symbol (imports + signatures).
   - `mode: "signature"`: find functions by signature pattern (e.g., `"-> Result<"`).
2. `summarize(path)` — understand files and symbols without reading source code.
   - File path (e.g., `"src/main.rs"`): complete file overview (declarations, imports, counts).
   - Glob pattern (e.g., `"src/mcp/*.rs"`): batch summaries for multiple files.
   - Symbol name (no `/`, e.g., `"Cache"`): full interface details (signature, doc comment, relationships).
   - `scope: "public"`: show only public API surface.
3. `read(path, symbol?)` — read source code by **symbol name** or explicit line range. Cap: 200 lines. Use `symbols` array to read multiple in one call (500 line cap). Use `collapse: true` to fold nested bodies.

With `--all-tools`, all 23 granular tools are also exposed:

4. `lookup_symbol` — look up a symbol by exact or partial name. Returns declaration details.
5. `list_declarations` — list all declarations in a file, optionally filtered by kind.
6. `search_signatures` — find functions by signature pattern (e.g., `"-> Result<"`).
7. `search_relevant` — multi-signal relevance search across paths, names, signatures, and docs.
8. `get_tree` — see directory/file layout. Use `path` param to scope to a subtree.
9. `get_file_summary` — get a single file's overview (declarations, imports, counts).
10. `read_source` — read source code by symbol name or line range (granular version of `read`).
11. `batch_file_summaries` — get summaries for multiple files in one call via glob pattern.
12. `get_file_context` — understand a file's reverse dependencies (who imports it) and related files (tests, siblings).
13. `get_callers` — find who references a symbol (imports + call sites).
14. `get_public_api` — list public declarations for a file or directory.
15. `explain_symbol` — full interface details for a symbol (signature, doc comment, relationships).
16. `get_token_estimate` — before deciding to `Read` a file, check how many tokens it costs. Supports `directory`, `glob`, or `symbol` for bulk/targeted estimation.
17. `get_related_tests` — find test functions for a symbol by naming convention and file association.
18. `get_diff_summary` — get structural changes since a git ref or GitHub PR number. Shows added/removed/modified declarations without reading full diffs.
19. `get_hotspots` — get the most complex functions/methods ranked by composite score.
20. `get_health` — get codebase health summary: aggregate complexity, documentation coverage, test ratio, hottest files.
21. `get_type_flow` — track where a type flows across function boundaries.
22. `get_dependency_graph` — get file-level or symbol-level dependency graph (DOT, Mermaid, JSON).
23. `get_stats` — codebase statistics: file count, line count, language breakdown.
24. `get_imports` — list import statements for a file.
25. `list_workspace_members` — list detected workspace members (Cargo, npm, Go workspaces).
26. `regenerate_index` — re-index after code changes. Updates INDEX.md, refreshes in-memory index, and reports what changed (delta).

#### Wiki tools (available when built with `--features wiki`)

If a wiki has been generated (`indxr wiki generate`), these tools are available automatically:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bahdotsh/indxr](https://github.com/bahdotsh/indxr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
