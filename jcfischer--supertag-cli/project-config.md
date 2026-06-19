---
trigger: always_on
description: description: Complete Tana integration via MCP. USE WHEN user mentions Tana, tana search, tana notes, my notes, my knowledge base, find in Tana, create in Tana, OR needs to search, query, or write to Tana workspace. Provides full-text search, semantic search, node creation, and workspace management.
---

# Supertag CLI Skill

---
name: supertag
description: Complete Tana integration via MCP. USE WHEN user mentions Tana, tana search, tana notes, my notes, my knowledge base, find in Tana, create in Tana, OR needs to search, query, or write to Tana workspace. Provides full-text search, semantic search, node creation, and workspace management.
---

## Overview

Supertag CLI provides complete Tana workspace integration through:
- **MCP Server** (`supertag-mcp`) - AI tool integration for Claude, ChatGPT, Cursor, etc.
- **CLI** (`supertag`) - Command-line queries, writes, and management
- **Webhook Server** - HTTP API for automation and Tana Commands

## MCP Tools Reference

### Progressive Disclosure (Start Here)

The MCP server supports progressive disclosure - a two-tier tool discovery pattern that reduces upfront token cost from ~2000 tokens to ~1000 tokens.

**Workflow:**
1. Call `tana_capabilities` to get a lightweight overview of all tools
2. Call `tana_tool_schema` to load full schemas for specific tools you need
3. Execute tools with validated parameters

### tana_capabilities
Get a lightweight overview of available tools, categorized by function.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `category` | string | No | Filter to specific category (query, explore, transcript, mutate, system) |

**Categories:**
- **query**: tana_search, tana_semantic_search, tana_tagged, tana_field_values, tana_batch_get, tana_query, tana_timeline, tana_recent, tana_table
- **explore**: tana_node, tana_related, tana_stats, tana_supertags, tana_supertag_info
- **transcript**: tana_transcript_list, tana_transcript_show, tana_transcript_search
- **mutate**: tana_create, tana_batch_create, tana_update_node, tana_tag_add, tana_tag_remove, tana_create_tag, tana_set_field, tana_set_field_option, tana_trash_node, tana_done, tana_undone
- **system**: tana_sync, tana_cache_clear, tana_capabilities, tana_tool_schema

**Example:**
```
What tools does the Tana MCP server provide?
Show me query tools for searching content
```

### tana_tool_schema
Load the full JSON schema for a specific tool. Use after `tana_capabilities` to get detailed parameter information.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tool` | string | Yes | Tool name (e.g., "tana_search") |

**Example:**
```
Get the full schema for tana_search
What parameters does tana_create accept?
```

### tana_search
Full-text search across Tana workspace using FTS5 indexing.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `limit` | number | No | Max results (default: 20) |
| `includeAncestor` | boolean | No | Include containing project/meeting context (default: true) |
| `createdAfter` | string | No | Filter by creation date (YYYY-MM-DD) |
| `createdBefore` | string | No | Filter by creation date |
| `updatedAfter` | string | No | Filter by update date |
| `updatedBefore` | string | No | Filter by update date |
| `workspace` | string | No | Workspace alias (default: main) |
| `select` | array | No | Fields to include in response (e.g., ["id", "name", "tags"]) |

**Example:**
```
Search my Tana for "authentication implementation"
Find notes about API design created after 2025-01-01
```

### tana_semantic_search
Vector similarity search using embeddings. Finds conceptually related content without exact keyword matches.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Natural language query |
| `limit` | number | No | Max results (default: 20) |
| `minSimilarity` | number | No | Threshold 0-1 (higher = stricter) |
| `includeContents` | boolean | No | Include full node details |
| `includeAncestor` | boolean | No | Include ancestor context (default: true) |
| `depth` | number | No | Child traversal depth (0-3) |
| `workspace` | string | No | Workspace alias |
| `select` | array | No | Fields to include in response (e.g., ["nodeId", "name", "similarity"]) |

**Example:**
```
Find notes semantically related to "knowledge management systems"
Search for concepts similar to "distributed architecture"
```

### tana_tagged
Find all nodes with a specific supertag applied.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| tagname` | string | Yes | Supertag name (e.g., "todo", "meeting") |
| `limit` | number | No | Max results (default: 20) |
| `orderBy` | string | No | Sort order (default: "created") |
| `caseInsensitive` | boolean | No | Case-insensitive matching |
| `createdAfter` | string | No | Filter by creation date |
| `createdBefore` | string | No | Filter by creation date |
| `workspace` | string | No | Workspace alias |
| `select` | array | No | Fields to include in response (e.g., ["id", "name", "created"]) |

**Example:**
```
Find all my todos in Tana

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jcfischer/supertag-cli](https://github.com/jcfischer/supertag-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
