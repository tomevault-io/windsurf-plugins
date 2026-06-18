---
trigger: always_on
description: Semantic code search MCP server. Monorepo: `packages/shared` (shared utilities), `packages/server` (MCP server + HTTP API), `packages/cli` (CLI tool), `packages/indexer` (automated repo indexer), and `packages/ollama` (custom Ollama Docker image).
---

# paparats-mcp

Semantic code search MCP server. Monorepo: `packages/shared` (shared utilities), `packages/server` (MCP server + HTTP API), `packages/cli` (CLI tool), `packages/indexer` (automated repo indexer), and `packages/ollama` (custom Ollama Docker image).

## IDs

Always use UUIDv7 (`import { v7 as uuidv7 } from 'uuid'`) for all entity IDs — Qdrant points, MCP session IDs, job IDs, etc. Never use `randomUUID()` or auto-increment. UUIDv7 is time-ordered, which matters for Qdrant and debugging.

## Architecture

- **Group** = Qdrant collection with `paparats_` prefix (e.g. group `my-app` → collection `paparats_my-app`). `toCollectionName()`/`fromCollectionName()` helpers in `indexer.ts` handle the prefix. Projects in the same group share a collection. `project` field in payload filters within a group.
- **`.paparats.yml`** = per-project config. Server reads it on demand via `readConfig()` / `resolveProject()`.
- **Server is stateless** — no hardcoded project list. Projects register via `POST /api/index`.
- **Qdrant client**: All `QdrantClient` instances must be created via `createQdrantClient({ url, apiKey?, timeout? })` from `indexer.ts`. This helper resolves the correct port from the URL protocol (HTTPS → 443, HTTP → 6333) — the JS client defaults to 6333 which breaks Qdrant Cloud. `QDRANT_API_KEY` env var → passed as `apiKey`. CLI: `--qdrant-api-key`. Docker Compose generator passes it via `${QDRANT_API_KEY}` env var substitution.
- **Embedding model**: `jina-code-embeddings` is a local Ollama alias for `jinaai/jina-code-embeddings-1.5b-GGUF`, registered via Modelfile. Not in Ollama registry.

## TypeScript conventions

- ESM only (`"type": "module"`). Imports use `.js` extension: `import { Foo } from './foo.js'`
- `strict: true`, `noUncheckedIndexedAccess: true` — always handle `T | undefined` from array/object indexing
- Target ES2022, module Node16
- Build: `yarn build` (runs `tsc` in each package)
- Lint: `yarn lint`, format: `yarn prettier`

## Module structure

**packages/shared**

- `path-validation.ts` — `validateIndexingPaths()` — rejects absolute paths and path traversal in `indexing.paths` (used by server and CLI)
- `exclude-patterns.ts` — `normalizeExcludePatterns()` — bare dir names (e.g. `node_modules`) become `**/node_modules/**` for glob
- `gitignore.ts` — `createGitignoreFilter()` (per-file checks), `filterFilesByGitignore()` (bulk filter) — used by CLI collectProjectFiles, server indexer, watch
- `language-excludes.ts` — `LANGUAGE_EXCLUDE_DEFAULTS`, `getDefaultExcludeForLanguages()` — per-language exclude patterns

**packages/server/src/**

| Module                    | Responsibility                                                                                                                                                                   |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `types.ts`                | Shared interfaces — all type definitions live here                                                                                                                               |
| `lib.ts`                  | Public library entry point — all re-exports for programmatic use (imported by `index.ts`, used by `@paparats/indexer`)                                                           |
| `config.ts`               | `.paparats.yml` reader, 11 built-in language profiles, `loadProject()`, `detectLanguages()`, `autoProjectConfig()`                                                               |
| `app.ts`                  | Express app factory (`createApp()`), HTTP API routes, `withTimeout()`, `sanitizeForLog()`                                                                                        |
| `index.ts`                | Server bootstrap — starts HTTP server, wires components, graceful shutdown                                                                                                       |
| `ast-chunker.ts`          | AST-based code chunking via tree-sitter — groups small nodes, splits large ones recursively                                                                                      |
| `chunker.ts`              | Regex-based code splitting (fallback) — 4 strategies (blocks, braces, indent, fixed)                                                                                             |
| `ast-symbol-extractor.ts` | AST-based symbol extraction — `extractSymbolsForChunks()` (defines/uses per chunk, 10 languages)                                                                                 |
| `ast-queries.ts`          | Tree-sitter S-expression query patterns per language                                                                                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBazylchuk/paparats-mcp](https://github.com/IBazylchuk/paparats-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
