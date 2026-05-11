---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides tools to search and retrieve Magento 2 GraphQL API documentation. It parses 350+ local markdown files with YAML frontmatter, extracts GraphQL schema elements, indexes content in SQLite with FTS5, and exposes 8 tools via FastMCP over STDIO.

## Documentation Source

### Required: Adobe Commerce GraphQL Documentation

The MCP server requires access to the official Adobe Commerce GraphQL documentation markdown files from the [AdobeDocs/commerce-webapi](https://github.com/AdobeDocs/commerce-webapi) repository.

**Clone the documentation repository:**

```bash
git clone https://github.com/AdobeDocs/commerce-webapi.git
```

The GraphQL documentation is located at: `commerce-webapi/src/pages/graphql/`

**This directory contains:**
- 350+ markdown files with YAML frontmatter
- Structured in categories: schema/, tutorials/, develop/, usage/, payment-methods/
- Total code examples: 963 blocks (GraphQL, JSON, PHP, Bash)
- GraphQL schema elements: 51 (queries, mutations, types, interfaces)

### Configuration Options

The server looks for documentation in this priority order (implemented in `config.py:get_docs_path()`):

1. **Environment Variable** (highest priority)
   ```bash
   export MAGENTO_GRAPHQL_DOCS_PATH="/path/to/commerce-webapi/src/pages/graphql"
   ```
   - Validated on startup - will error if path doesn't exist
   - Use absolute paths for best results

2. **Symlink in Project** (`./data/` directory - recommended for development)
   ```bash
   cd magento-graphql-docs-mcp
   ln -s /path/to/commerce-webapi/src/pages/graphql data
   ```
   - Checks if `data/` exists and contains `.md` files
   - Can be symlink or actual directory with markdown files
   - Resolved to absolute path automatically

3. **Sibling Directory** (automatic detection)
   ```
   projects/
   ├── magento-graphql-docs-mcp/  (this server)
   └── commerce-webapi/
       └── src/pages/graphql/     (documentation)
   ```
   - Automatically detected if commerce-webapi is cloned as sibling

**Important**: All paths are validated on startup. If no valid documentation is found, the server will fail with a helpful error message explaining the three setup options.

### Verification

Before running the server, verify documentation access:

```bash
# Check environment variable (if set)
echo $MAGENTO_GRAPHQL_DOCS_PATH
ls -la $MAGENTO_GRAPHQL_DOCS_PATH

# Check symlink (if used)
ls -la data/

# Expected structure:
# - index.md (GraphQL overview)
# - release-notes.md
# - schema/ (285 files - queries, mutations, types)
# - tutorials/ (13 files - checkout, orders, etc.)
# - develop/ (8 files - extending GraphQL)
# - usage/ (10 files - using GraphQL API)
# - payment-methods/ (11 files)
```

### Database Location

The SQLite database is stored at `~/.mcp/magento-graphql-docs/database.db` by default.

Override with:
```bash
export MAGENTO_GRAPHQL_DOCS_DB_PATH="/custom/path/database.db"
```

The database is automatically created on first run and only re-indexed when markdown files change (based on mtime).

## Core Architecture

### Three-Layer System

1. **Parser Layer** (`magento_graphql_docs_mcp/parser.py`)
   - Parses markdown files with YAML frontmatter using `python-frontmatter`
   - Extracts document metadata (title, description, keywords from frontmatter)
   - Extracts categories from file paths (e.g., "schema/products/queries" → category="schema", subcategory="products")
   - Extracts markdown structure (headers, code blocks)
   - Detects GraphQL elements in code blocks (queries, mutations, types, interfaces)
   - Builds searchable text combining all available text
   - Uses Pydantic models: `Document`, `CodeBlock`, `GraphQLElement`

2. **Ingestion Layer** (`magento_graphql_docs_mcp/ingest.py`)
   - File modification time tracking (only re-parse if files change)
   - Creates SQLite schema with 4 tables: documents, code_blocks, graphql_elements, metadata
   - FTS5 indexes on: documents.searchable_text and graphql_elements.searchable_text (trigram tokenization)
   - Bulk inserts for performance (350 documents, 963 code blocks, 51 GraphQL elements)
   - Clears existing data before re-ingestion (not incremental)

3. **Server Layer** (`magento_graphql_docs_mcp/server.py`)
   - FastMCP server with lifespan manager that triggers ingestion on startup
   - Error handling in lifespan: catches FileNotFoundError with helpful setup instructions
   - Exposes 8 tools over STDIO
   - Returns formatted markdown responses
   - Returns top K results for searches (configurable via MAGENTO_GRAPHQL_DOCS_TOP_K env var, default: 5)
   - Uses configurable constants for code preview lengths and field limits

### MCP Tools (8 Total)

1. **search_documentation**: FTS search on documents with category/subcategory/content_type filters
2. **get_document**: Direct lookup by file_path, returns full markdown content
3. **search_graphql_elements**: FTS search on graphql_elements with element_type filter
4. **get_element_details**: Lookup element by name with source document and code examples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [florinel-chis/magento-graphql-docs-mcp](https://github.com/florinel-chis/magento-graphql-docs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
