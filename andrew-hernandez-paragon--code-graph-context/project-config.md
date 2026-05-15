---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Code Graph Context is an MCP (Model Context Protocol) server that builds code graphs to provide rich context to LLMs. It parses TypeScript codebases using AST analysis (ts-morph), stores the graph in Neo4j with vector embeddings, and provides semantic search and graph traversal tools.

## Build & Development Commands

```bash
npm run build          # Compile TypeScript to dist/
npm run dev            # Watch mode compilation
npm run mcp            # Run MCP server: node dist/mcp/mcp.server.js
npm run lint           # ESLint with auto-fix
npm run format         # Prettier formatting
```

## Architecture

### Data Flow
```
TypeScript Project → AST Parser (ts-morph) → Graph Nodes/Edges → Neo4j + Vector Embeddings → MCP Tools
```

### Key Directories

- `src/mcp/` - MCP server entry point and tools
  - `mcp.server.ts` - Server initialization
  - `tools/` - 7 MCP tools (search_codebase, traverse_from_node, impact_analysis, etc.)
  - `handlers/` - Business logic for graph generation and traversal
- `src/core/` - Core business logic
  - `parsers/typescript-parser.ts` - Main AST parser (~1000 lines)
  - `config/schema.ts` - Core graph schema definitions
  - `config/nestjs-framework-schema.ts` - NestJS semantic patterns
  - `embeddings/` - OpenAI embeddings and NL-to-Cypher services
- `src/storage/neo4j/` - Neo4j driver and queries

### Dual-Schema System

The parser uses two schema layers:
1. **Core Schema** (AST-level): ClassDeclaration, MethodDeclaration, PropertyDeclaration, ImportDeclaration, etc.
2. **Framework Schema** (Semantic): Controller, Service, Module, Guard, Repository, etc. (NestJS patterns)

Nodes have both `coreType` (AST) and `semanticType` (framework interpretation).

### Multi-Project Support

The system supports multiple projects in a single Neo4j database through project isolation:

- **Project ID Format**: `proj_<12-hex-chars>` (e.g., `proj_a1b2c3d4e5f6`)
- **Auto-generation**: If not provided, projectId is generated deterministically from the project path
- **Explicit Override**: Pass `projectId` to `parse_typescript_project` to use a custom ID
- **Isolation**: All queries are automatically scoped to the project - nodes from different projects never interfere

**Usage in Tools:**
```typescript
// All query tools require projectId
search_codebase({ projectId: "proj_abc123...", query: "..." })
traverse_from_node({ projectId: "proj_abc123...", nodeId: "..." })
impact_analysis({ projectId: "proj_abc123...", nodeId: "..." })

// parse_typescript_project returns the resolved projectId
const result = await parse_typescript_project({ projectPath: "/path/to/project" });
// result.resolvedProjectId => "proj_a1b2c3d4e5f6"
```

### Migration from Pre-Multi-Project Versions

If upgrading from a version without multi-project support, note these breaking changes:

**Breaking Changes:**
- Node IDs now include projectId prefix (format: `proj_xxx:CoreType:hash`)
- All query tools now require `projectId` parameter
- Existing nodes in the database have old ID format and won't be accessible

**Migration Options:**

1. **Clear and Re-parse (Recommended)**
   ```bash
   # Clear the database and re-parse your project
   # The new projectId will be auto-generated from the project path
   ```

2. **Continue Without Multi-Project**
   - Not recommended - existing node IDs are incompatible
   - Queries will fail to find nodes with old ID format

**Note:** There is no automatic migration path. Existing graphs must be rebuilt to use the new ID format with projectId isolation.

### MCP Tools

| Tool | Purpose |
|------|---------|
| `search_codebase` | Primary tool — semantic search via vector embeddings |
| `traverse_from_node` | Follow-up exploration from a node ID |
| `impact_analysis` | Risk assessment — analyze dependencies before changes |
| `parse_typescript_project` | Build the graph from source code |
| `check_parse_status` | Poll async parsing job status |
| `natural_language_to_cypher` | Advanced — convert NL to Cypher queries (requires OpenAI) |
| `detect_dead_code` | Find unused exports, uncalled methods |
| `detect_duplicate_code` | Find structural and semantic duplicates |
| `list_projects` | List parsed projects |
| `test_neo4j_connection` | Diagnostic — verify Neo4j connectivity |
| `swarm_claim_task` | Claim a task from the queue |
| `swarm_release_task` | Release or abandon a claimed task |
| `swarm_advance_task` | Start or force-start a claimed task |
| `swarm_complete_task` | Mark task completed/failed |
| `swarm_post_task` | Post task to queue |
| `swarm_get_tasks` | Query tasks with filters |
| `swarm_pheromone` | Mark code nodes for coordination |
| `swarm_sense` | Query active pheromones |
| `swarm_cleanup` | Bulk delete pheromones/tasks |
| `swarm_message` | Direct agent-to-agent messaging |
| `session_save` | Save bookmark or note (unified) |
| `session_recall` | Restore bookmark or search notes (unified) |
| `cleanup_session` | Remove expired session data |
| `start_watch_project` | Watch for file changes |
| `stop_watch_project` | Stop watching |
| `list_watchers` | List active watchers |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrew-hernandez-paragon/code-graph-context](https://github.com/andrew-hernandez-paragon/code-graph-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
