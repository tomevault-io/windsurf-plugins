---
trigger: always_on
description: **Generated:** 2025-01-16 | **Commit:** 9f94822 | **Branch:** main
---

# AGENTS.md - AI Agent Guidelines for opencode-codebase-index

**Generated:** 2025-01-16 | **Commit:** 9f94822 | **Branch:** main

Semantic codebase indexing plugin for OpenCode. Hybrid TypeScript/Rust architecture:
- **TypeScript** (`src/`): Plugin logic, embedding providers, OpenCode tools
- **Rust** (`native/`): Tree-sitter parsing, usearch vectors, SQLite storage, BM25 inverted index, call graph extraction

## Build/Test/Lint

```bash
npm run build          # Build TS + Rust native module
npm run build:ts       # TypeScript only (tsup)
npm run build:native   # Rust only (cargo + napi)

npm run test:run       # All tests once
npm test               # Watch mode

npm run lint           # ESLint
npm run typecheck      # tsc --noEmit
```

### Single Test
```bash
npx vitest run tests/files.test.ts
npx vitest run -t "parseFile"
```

### Native Module (requires Rust)
```bash
cd native && cargo build --release && napi build --release --platform
```

## File Structure

```
src/
├── index.ts              # Plugin entry: exports tools + slash commands
├── mcp-server.ts         # MCP server: wraps Indexer for Cursor/Claude Code/Windsurf
├── cli.ts                # CLI entry point for MCP stdio transport
├── config/               # Config schema (Zod) + parsing
├── embeddings/           # Provider detection (auto/github/openai/google/ollama)
├── indexer/              # Core: Indexer class, delta tracking
├── git/                  # Branch detection from .git/HEAD
├── tools/                # OpenCode tool definitions (codebase_search, index_*)
├── utils/                # File collection, cost estimation, Logger
├── native/               # TS wrapper for Rust bindings
└── watcher/              # Chokidar file + git branch watcher

native/src/
├── lib.rs                # NAPI exports: parse_file, VectorStore, Database, InvertedIndex
├── parser.rs             # Tree-sitter parsing (14 languages: TS, JS, Python, Rust, Go, Java, C#, Ruby, Bash, C, C++, JSON, TOML, YAML)
├── chunker.rs            # Semantic chunking with overlap
├── store.rs              # usearch vector store (F16 quantization)
├── db.rs                 # SQLite: embeddings, chunks, branch catalog, symbols, call edges
├── call_extractor.rs     # Tree-sitter query-based call extraction (TS/JS, Python, Go, Rust)
├── inverted_index.rs     # BM25 keyword search
├── hasher.rs             # xxhash content hashing
└── types.rs              # Shared types (Language enum with from_string)

tests/                    # Vitest tests (30s timeout for native ops)
commands/                 # Slash command definitions (/search, /find, /index)
skill/                    # OpenCode skill guidance
```

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Add embedding provider | `src/embeddings/detector.ts` + `provider.ts` |
| Modify indexing logic | `src/indexer/index.ts` (Indexer class) |
| Add OpenCode tool | `src/tools/index.ts` |
| Change parsing behavior | `native/src/parser.rs` |
| Modify vector storage | `native/src/store.rs` |
| Add database operation | `native/src/db.rs` + expose in `lib.rs` |
| Add slash command | `commands/` + register in `src/index.ts` config() |

| Add/modify MCP tool | `src/mcp-server.ts` (createMcpServer) |
| Modify call graph extraction | `native/src/call_extractor.rs` + query files in `native/queries/` |
| Add call graph language | `native/queries/<lang>-calls.scm` + update `call_extractor.rs` |
## CODE MAP

### TypeScript Exports (`src/index.ts`)
| Symbol | Type | Purpose |
|--------|------|---------|
| `default` | Plugin | Main entry: returns tools + config callback |
| `codebase_search` | Tool | Semantic search by meaning (returns full code content) |
| `codebase_peek` | Tool | Semantic search returning metadata only (file, line, name) - saves tokens |
| `find_similar` | Tool | Find code similar to a given snippet (duplicate detection, pattern discovery) |
| `index_codebase` | Tool | Trigger indexing (force/estimate/verbose) |
| `index_status` | Tool | Check index health |
| `index_health_check` | Tool | GC orphaned embeddings/chunks |
| `index_metrics` | Tool | Get performance metrics (requires debug.enabled + debug.metrics) |
| `index_logs` | Tool | Get debug logs (requires debug.enabled) |
| `call_graph` | Tool | Query call graph for callers/callees of functions |


### MCP Server Exports (`src/mcp-server.ts`)
| Symbol | Type | Purpose |
|--------|------|---------|
| `createMcpServer` | fn | Creates MCP Server with 9 tools + 4 prompts, lazy Indexer init |

### CLI Entry (`src/cli.ts`)
| Symbol | Type | Purpose |
|--------|------|---------|
| `main` | fn | Parses --project/--config args, starts stdio transport, handles shutdown |
### Rust NAPI Exports (`native/src/lib.rs`)
| Symbol | Type | Purpose |
|--------|------|---------|
| `parse_file` | fn | Parse single file → CodeChunk[] |
| `parse_files` | fn | Parallel multi-file parsing |
| `hash_content` | fn | xxhash string |
| `hash_file` | fn | xxhash file contents |
| `VectorStore` | class | usearch wrapper (add/addBatch/search/save/load) |
| `Database` | class | SQLite: embeddings, chunks, branches, metadata (includes batch methods) |
| `InvertedIndex` | class | BM25 keyword search |

### Database Batch Methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Helweg/opencode-codebase-index](https://github.com/Helweg/opencode-codebase-index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
