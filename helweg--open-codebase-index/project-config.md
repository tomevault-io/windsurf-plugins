---
trigger: always_on
description: **Updated:** 2026-08-02 | **Commit:** 32160f8 | **Branch:** main | **Version:** 0.22.3
---

# AGENTS.md - AI Agent Guidelines for open-codebase-index

**Updated:** 2026-08-02 | **Commit:** 32160f8 | **Branch:** main | **Version:** 0.22.3

Semantic codebase indexing for OpenCode, MCP hosts, Pi, Claude, Codex, and Jcode. repo uses hybrid TypeScript/Rust architecture:

- **TypeScript** (`src/`): host adapters, tool orchestration, indexing, retrieval, embedding providers, config, evaluation, and watchers
- **Rust** (`native/`): tree-sitter parsing, semantic chunking, usearch vectors, SQLite persistence, BM25, call graphs, and graph analytics

## Build, Test, and Lint

```bash
npm run build          # Build TypeScript and the Rust native module
npm run build:ts       # TypeScript bundle plus built-CLI smoke test
npm run build:native   # Rust/NAPI module for the current platform

npm run test:run       # Full Vitest suite once; pretest rebuilds native code
npm test               # Vitest watch mode
npm run test:coverage  # Coverage run; pretest rebuilds native code

npm run lint           # ESLint over src/
npm run typecheck      # tsc --noEmit
```

### Run a Single Test

```bash
npx vitest run tests/files.test.ts
npx vitest run -t "parseFile"
```

When Rust code changes, rebuild native module before running targeted tests that bypass `npm run test:run`

```bash
npm run build:native
# Equivalent low-level command:
cd native && cargo build --release && napi build --release --platform
```

 full PR validation gate is:

```bash
npm run build && npm run typecheck && npm run lint && npm run test:run
```

## Architecture and File Structure

```text
src/
├── index.ts                    # Thin OpenCode facade; re-exports adapters/opencode
├── mcp-server.ts               # Thin MCP facade; re-exports createMcpServer
├── cli.ts                      # CLI facade for MCP, eval, and visualization commands
├── adapters/
│   ├── opencode.ts             # OpenCode plugin composition and hooks
│   ├── opencode/               # OpenCode tool and PR-impact adapters
│   ├── mcp/                    # MCP CLI, server, tools, prompts, and shared schemas
│   └── pi/                     # Pi extension and call-graph adapters
├── tools/
│   ├── operations.ts           # Shared host-neutral tool operations
│   ├── operation-runtime.ts    # Shared operation runtime/context
│   ├── contracts.ts            # Shared request/result contracts
│   ├── execute-common.ts       # Common execution helpers
│   ├── context*.ts             # Context routing, retrieval, and evidence packing
│   └── tool-names.ts           # Canonical and host-specific public tool names
├── indexer/
│   ├── index.ts                # Indexer orchestration
│   ├── search-ranking.ts       # Hybrid fusion, filtering, diversity, assembly
│   ├── definition-ranking.ts   # Definition-oriented evidence ranking
│   ├── embedding-batches.ts    # Embedding batching, retry state, vector pooling
│   └── call-graph-constants.ts # Shared declaration chunk-type rules
├── native/                     # Focused TypeScript wrappers over the NAPI binding
├── config/                     # Host-aware config schema, merging, paths, and validation
├── embeddings/                 # Provider detection and implementations
├── git/                        # Branch resolution and branch-index materialization
├── watcher/                    # File and Git branch watchers
├── eval/                       # Retrieval evaluation CLI, datasets, metrics, and reports
├── rerank/                     # Optional external reranking
├── utils/                      # Files, paths, logging, metrics, power state, and helpers
├── identity-catalog.json       # Current/future product and package identities
└── package-metadata.ts         # Runtime package metadata helpers

native/src/
├── lib.rs                      # NAPI facade and exports
├── bindings/
│   └── database.rs             # NAPI Database wrapper methods
├── db.rs                       # Core SQLite database implementation
├── db/call_graph.rs            # Call-graph persistence and query operations
├── parser.rs                   # Tree-sitter parsing
├── chunker.rs                  # Semantic chunking with overlap
├── call_extractor.rs           # Query-based call extraction
├── community.rs                # Community detection and centrality algorithms
├── store.rs                    # usearch vector storage
├── inverted_index.rs           # BM25 keyword index
├── hasher.rs                   # xxhash content hashing
└── types.rs                    # Shared native types and language mapping

native/queries/                 # Tree-sitter call queries by language
tests/                          # Vitest integration and unit tests
benchmarks/                     # Native and retrieval benchmarks/evaluation fixtures
commands/                       # OpenCode slash command definitions
skill/                          # OpenCode skill guidance
docs/                           # Installation, configuration, tools, migration docs
```

See `ARCHITECTURE.md` for data flow and design details.

## Where to Look

| Task | Primary location |
|---|---|
| Add or change an OpenCode integration | `src/adapters/opencode.ts`, `src/adapters/opencode/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Helweg/open-codebase-index](https://github.com/Helweg/open-codebase-index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
