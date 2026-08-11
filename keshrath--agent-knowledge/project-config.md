---
trigger: always_on
description: Layered architecture — single `server.ts` handles MCP tools, separate `dashboard.ts` for HTTP/WebSocket:
---

# agent-knowledge

## Architecture

Layered architecture — single `server.ts` handles MCP tools, separate `dashboard.ts` for HTTP/WebSocket:

```
scripts/
  tree-sitter-extract.mjs  Standalone CLI for zero-token code structure extraction
  tree-sitter-lang.mjs     Per-language tree-sitter query definitions (8 languages)
  setup.js                 Automated setup wizard (MCP server + hooks + skill install)
  hooks/                   Claude Code lifecycle hooks
skills/
  knowledge-ingest/
    SKILL.md               Claude Code skill file
src/
  server.ts             MCP server, 6 tool definitions, request routing
  dashboard.ts          HTTP + WebSocket server, REST API, file watcher
  index.ts              Entry point (MCP stdio + dashboard auto-start)
  types.ts              KnowledgeConfig, getConfig(), getVersion()
  package-meta.ts       Cached name/version from package.json
  validate.ts           ValidationError class, input validation
  version.ts            Re-exports getVersion from types.ts
  knowledge/
    store.ts            Markdown CRUD, frontmatter parsing, path traversal protection
    search.ts           TF-IDF search over knowledge entries with regex fallback
    git.ts              git pull/push/sync with timeouts
    distill.ts          Session auto-distillation with secrets scrubbing
    graph.ts            Knowledge graph — edges table, link/unlink/traverse (BFS)
    scoring.ts          Confidence/decay scoring — entry_scores table, auto-promotion
    consolidate.ts      Memory consolidation — TF-IDF duplicate detection, cluster grouping
    reflect.ts          Reflection cycle — surfaces unconnected entries, generates prompts
    analyze.ts          Graph analysis — god nodes, bridges, gaps, knowledge brief
  sessions/
    parser.ts           Multi-format session parsing with mtime-based cache
    indexer.ts           Background indexing for sessions
    search.ts           TF-IDF ranked search with 60s global index cache
    scopes.ts           Search scopes (errors, plans, configs, tools, files, decisions)
    summary.ts          Session summaries, topic extraction, file path detection
    adapters/
      index.ts          SessionAdapter interface, adapter registry, auto-init
      opencode.ts       OpenCode adapter (SQLite database)
  search/
    tfidf.ts            TF-IDF scoring engine (tokenizer, stopwords, index)
    fuzzy.ts            Levenshtein distance, sliding window fuzzy matching
    excerpt.ts          Search result excerpt generation
    types.ts            SearchResult, SearchOptions interfaces
  embeddings/
    index.ts            Embedding provider registry
    factory.ts          Provider factory (auto-detect available providers)
    types.ts            EmbeddingProvider interface
    claude.ts           Claude/Voyage embeddings
    openai.ts           OpenAI embeddings
    gemini.ts           Gemini embeddings
    local.ts            Local embedding fallback
  vectorstore/
    index.ts            Vector store facade
    store.ts            SQLite-backed vector storage with cosine similarity
    chunker.ts          Document chunking for embedding
  ui/
    index.html          Dashboard SPA
    styles.css          MD3 design tokens (light + dark)
    app.js              Client-side vanilla JS (WebSocket, tabs, rendering)
```

## UI / Dashboard

- **Icons**: Material Symbols Outlined (via Google Fonts CSS). No emojis.
- **Fonts**: Inter (UI text), JetBrains Mono (code/data)
- **Theme**: Light/dark toggle
- **Design tokens**: CSS custom properties (`--bg`, `--accent`, `--border`, `--shadow-*`, etc.)
- **Accent color**: `#5d8da8`
- **Port**: 3423 (configurable via `KNOWLEDGE_PORT`)

## Code Style

- **TypeScript** with strict mode, ES modules
- **Imports**: use `.js` extensions (TypeScript NodeNext convention)
- **Naming**: `camelCase` for functions/variables, `PascalCase` for types/classes, `UPPER_SNAKE` for constants
- **Async**: use `async`/`await` over raw promises
- **Error handling**: throw descriptive errors, catch and return MCP-formatted errors in tool handlers
- **No external formatters** — match existing code style
- **ESLint + Prettier** enforced via lint-staged (husky pre-commit)

## Versioning

- Version lives in `package.json` and is read at runtime via `version.ts`
- Never hardcode version strings

## Build & Test

```
npm run build      # tsc + copy UI files to dist/
npm test           # vitest (unit tests)
npm run check      # typecheck + lint + format + test
npm run dev        # watch mode (tsc --watch)
```

## Key APIs

- **MCP (6 tools, all exposed to the LLM):**
  - `knowledge` — actions `list`, `read`, `write`, `delete`, `sync`, `wakeup`.
  - `knowledge_search` — hybrid general OR scoped (via `scope`); response shape `{mode, sessions, knowledge}`. v1.8 knobs: `mmr`, `mmr_lambda`, `category_mode` (default `boost`), `explain`.
  - `knowledge_session` — `list`, `get`, `summary`.
  - `knowledge_graph` — `link`, `unlink`, `invalidate`, `list`, `traverse`, `bulk_link`, `unlink_by_origin`.
  - `knowledge_analyze` — `consolidate`, `reflect`, `god_nodes`, `bridges`, `gaps`, `brief`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keshrath/agent-knowledge](https://github.com/keshrath/agent-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
