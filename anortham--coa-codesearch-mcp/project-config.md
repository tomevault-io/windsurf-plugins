---
trigger: always_on
description: **SEMANTIC SEARCH IS A CORE FEATURE - NEVER SUGGEST REMOVING IT**
---

# CodeSearch MCP Server - Developer Guide

## ⚠️ NON-NEGOTIABLE REQUIREMENTS

**SEMANTIC SEARCH IS A CORE FEATURE - NEVER SUGGEST REMOVING IT**
- 3-tier search architecture: SQLite → Lucene → Semantic (vec0/HNSW)
- Semantic search is the tentpole feature - focus on making it FAST, not removing it
- If performance is an issue, optimize the implementation, don't remove functionality

## 🎯 Quick Reference

Lucene.NET-powered code search with julie-codesearch native type extraction. Local workspace indexing with cross-platform support.

**Version**: 4b32d04 | **Status**: Production Ready | **Performance**: 117 files/sec | **Framework**: julie-codesearch Rust CLI (25 languages)

### Core Tools (14 available) - **Now with Smart Defaults!** 🎯

```bash
# Essential Search
text_search         # Full-text code search with CamelCase tokenization
symbol_search       # Find classes, methods, interfaces by name
goto_definition     # Jump to exact symbol definitions
find_references     # Find ALL usages (critical before refactoring)

# File Discovery
search_files        # 🆕 Unified file/directory search (replaces file_search + directory_search)
recent_files        # Recent modifications (great for context)

# Advanced Search
line_search         # Line-by-line search (replaces grep/rg)
search_and_replace  # Bulk find & replace with preview + fuzzy matching
trace_call_path     # Hierarchical call chain analysis with semantic bridging

# Refactoring
smart_refactor      # AST-aware symbol renaming (byte-offset precision)

# Code Editing
edit_lines          # 🆕 Unified line editing (insert/replace/delete - replaces 3 tools)

# Semantic Analysis
get_symbols_overview # Extract all symbols from files (classes, methods, etc.)
find_patterns       # Detect code patterns and quality issues

# System
index_workspace     # Build/update search index (REQUIRED FIRST)
```

**Note:** Tools now have smart defaults - most calls need only 1-2 parameters!
**Replaced:** `insert_at_line`, `replace_lines`, `delete_lines` → use `edit_lines`
**Replaced:** `file_search`, `directory_search` → use `search_files`
**Removed:** `batch_operations` (obsolete with MCP parallel tool calls), `similar_files` (never implemented), `SearchType` parameter (use `SearchMode` instead)

## 🚨 Development Workflow

**Code Changes:**

1. Exit Claude Code completely
2. `dotnet build -c Debug` (Debug mode recommended)
3. Restart Claude Code
4. ⚠️ **Testing before restart shows OLD CODE**

**Never run:** `dotnet run -- stdio` (creates orphaned processes)

## 📝 Tool Parameter Documentation

**IMPORTANT: XML Comments are Primary, [Description] is Fallback**

When documenting tool parameters:
- **XML `/// <summary>` comments**: Primary source for MCP tool descriptions - this is what Claude sees
- **`[Description("...")]` attribute**: Fallback only if XML comments are missing
- **Always document in BOTH places** for consistency, but XML comments take precedence

Example:
```csharp
/// <summary>
/// The refactoring operation to perform.
/// Valid operations: rename_symbol, extract_to_file, move_symbol_to_file, extract_interface
/// </summary>
[Description("The refactoring operation to perform: rename_symbol, extract_to_file, move_symbol_to_file, extract_interface")]
public required string Operation { get; set; }
```

## 🔍 Usage Essentials - **Minimal Parameters!** ✨

**Always start with:**

```bash
# Index with just the path (or omit for current directory)
mcp__codesearch__index_workspace --workspacePath "."
```

**Common patterns (now with smart defaults):**

```bash
# Search - just the query! (workspace defaults to current directory)
mcp__codesearch__text_search --query "class UserService"

# Verify types - just the symbol name!
mcp__codesearch__goto_definition --symbol "UserService"

# Check impact - just the symbol!
mcp__codesearch__find_references --symbol "UpdateUser"

# Find files - just the pattern! (defaults to files, current workspace)
mcp__codesearch__search_files --pattern "*.cs"
mcp__codesearch__search_files --pattern "**/*.csproj"

# Find directories - specify resourceType
mcp__codesearch__search_files --pattern "test*" --resourceType "directory"

# Edit code - operation + line + content
mcp__codesearch__edit_lines --filePath "User.cs" --operation "insert" --startLine 42 --content "// TODO"
mcp__codesearch__edit_lines --filePath "User.cs" --operation "replace" --startLine 10 --endLine 15 --content "refactored code"
mcp__codesearch__edit_lines --filePath "User.cs" --operation "delete" --startLine 20 --endLine 25
```

### ⚡ Before vs After

**Before (8+ parameters):**
```bash
text_search --workspacePath "." --query "UserService" --searchMode "auto" --searchType "standard" --caseSensitive false --maxTokens 8000 --noCache false --documentFindings false --autoDetectPatterns false
```

**After (1 parameter!):**
```bash
text_search --query "UserService"  # All other params have smart defaults!
```

## 🏗️ Architecture

**Index Storage**: `.coa/codesearch/indexes/{workspace-hash}/` (local per workspace)
- `lucene/` - Lucene.NET full-text search index
- `db/` - SQLite canonical symbol database (workspace.db)
- `vectors/` - HNSW semantic search index (julie-semantic)

**Logs**: `.coa/codesearch/logs/` (workspace-specific logging)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anortham/coa-codesearch-mcp](https://github.com/anortham/coa-codesearch-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
