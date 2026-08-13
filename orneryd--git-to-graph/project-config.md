---
trigger: always_on
description: Build a standalone Vite/React/TypeScript app that connects to a local NornicDB instance and provides an interactive D3.js force-directed graph visualization of code repositories indexed by `git-to-graph`.
---

# Graph Explorer — Code Knowledge Graph Visualizer

Build a standalone Vite/React/TypeScript app that connects to a local NornicDB instance and provides an interactive D3.js force-directed graph visualization of code repositories indexed by `git-to-graph`.

## What this app does

After an engineer runs `g2g index .` (the git-to-graph CLI), a temporal code knowledge graph is loaded into NornicDB with commits, files, symbols, and relationships — all versioned with `valid_from`/`valid_to` windows. This UI lets the engineer visually explore that graph and scrub through commit history with a slider to see the codebase at any point in time.

## NornicDB Connection

Default local connection (no env config needed for dev):

- **HTTP API**: `http://localhost:7474`
- **Auth**: Basic auth, `admin` / `password`
- **Default database**: `nornic`

Two API surfaces are used:

1. **Cypher endpoint**: `POST /db/{database}/tx/commit` — for all data queries
2. **Graph REST API**: `POST /nornicdb/graph/{database}/neighborhood|temporal|diff` — for graph traversal

## Tech Stack

Match the NornicDB UI conventions exactly:

- **Vite** with `@vitejs/plugin-react`
- **React 19** (`react@^19.2.5`, `react-dom@^19.2.5`)
- **TypeScript**
- **D3.js** (`d3-force`, `d3-zoom`, `d3-selection`, `d3-scale`, `d3-color`) for graph rendering into SVG
- **Tailwind CSS 4** (`tailwindcss@^4.2.2`, `@tailwindcss/postcss@^4.2.2`)
- **zustand** (`zustand@^5.0.12`) for state management
- **lucide-react** (`lucide-react@^1.8.0`) for icons

## API Client

Adapt the NornicDB client pattern. The client needs two methods:

### executeCypher(statement, parameters?, database?)

```typescript
// POST /db/{database}/tx/commit
// Body: { statements: [{ statement, parameters }] }
// Response: { results: [{ columns: string[], data: [{ row: unknown[], meta: unknown[] }] }], errors?: [...] }

interface CypherResponse {
  results: Array<{
    columns: string[];
    data: Array<{ row: unknown[]; meta: unknown[] }>;
  }>;
  errors?: Array<{ code: string; message: string }>;
}
```

### Graph REST API calls

```typescript
// All POST, all return the same shape:
interface GraphPayload {
  nodes: Array<{
    id: string;
    labels: string[];
    properties: Record<string, unknown>;
    status?: string; // "added" | "removed" | "changed" | "" (only on diff)
  }>;
  edges: Array<{
    id: string;
    source: string;
    target: string;
    type: string;
    properties?: Record<string, unknown>;
    semantic?: boolean;
    status?: string; // same as above
  }>;
  meta: {
    database: string;
    generated_from: string;
    depth?: number;
    as_of?: string;
    compare_to?: string;
    node_count: number;
    edge_count: number;
    truncated: boolean;
  };
}

// POST /nornicdb/graph/{database}/neighborhood
// Body: { node_ids: string[], depth?: number, limit?: number, labels?: string[], relationship_types?: string[] }

// POST /nornicdb/graph/{database}/temporal  
// Body: { node_ids: string[], as_of: string (ISO 8601 datetime) }

// POST /nornicdb/graph/{database}/diff
// Body: { node_ids: string[], as_of: string, compare_to?: string }
```

Use Basic auth header on all requests: `Authorization: Basic ${btoa('admin:password')}`.

## Cypher Queries

All queries are designed to use indexed fields. Place these in a dedicated `src/api/queries.ts` file as parameterized template functions.

### Q1: List indexed repositories
```cypher
MATCH (ck:RepositoryKey)-[:HAS_STATE]->(cs:RepositoryState)
WHERE cs.valid_to IS NULL
RETURN ck.entity_id AS repo_id, cs.value_json AS info
```

### Q2: List commits ordered by time
```cypher
MATCH (c:Commit)
RETURN c.hash AS hash, c.timestamp AS timestamp, c.actor AS actor
ORDER BY c.timestamp ASC
LIMIT $limit
```
Parameters: `{ limit: 500 }`

### Q3: Files alive at a commit timestamp
```cypher
MATCH (cs:CodeFileState)
WHERE cs.valid_from <= datetime($timestamp)
  AND (cs.valid_to IS NULL OR cs.valid_to > datetime($timestamp))
RETURN cs.state_id AS id, cs.code_key AS key, cs.value_json AS value
```
Parameters: `{ timestamp: "2025-01-15T00:00:00Z" }`

### Q4: Directories alive at a commit timestamp
```cypher
MATCH (cs:DirectoryState)
WHERE cs.valid_from <= datetime($timestamp)
  AND (cs.valid_to IS NULL OR cs.valid_to > datetime($timestamp))
RETURN cs.state_id AS id, cs.code_key AS key, cs.value_json AS value
```

### Q5: Containment edges at a commit timestamp
```cypher
MATCH (cs:ContainsEdgeState)
WHERE cs.valid_from <= datetime($timestamp)
  AND (cs.valid_to IS NULL OR cs.valid_to > datetime($timestamp))
RETURN cs.state_id AS id, cs.code_key AS key, cs.value_json AS value
```

### Q6: Symbols in a file at a commit timestamp (drill-down)
```cypher
MATCH (cs:CodeState)
WHERE cs.semantic_type IN ['function', 'method', 'class', 'type', 'struct', 'interface', 'variable', 'constant']
  AND cs.valid_from <= datetime($timestamp)
  AND (cs.valid_to IS NULL OR cs.valid_to > datetime($timestamp))
  AND cs.code_key CONTAINS $filePath
RETURN cs.state_id AS id, cs.code_key AS key, cs.value_json AS value, cs.semantic_type AS kind
```
Parameters: `{ timestamp: "...", filePath: "internal/model/model.go" }`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orneryd/git-to-graph](https://github.com/orneryd/git-to-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
