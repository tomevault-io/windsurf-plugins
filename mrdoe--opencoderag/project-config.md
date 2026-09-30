---
trigger: always_on
description: Local-first RAG plugin for semantic code search — tree-sitter chunking, LanceDB, hybrid retrieval
---


## Code Navigation

ALWAYS use OpenCodeRAG tools before reading or editing:
- **Search first** — `search_semantic(query)` instead of grep/glob. Optional args: `pathHints`, `languageHints`, `fileExtensions` (e.g. `[".ts"]`), `topK`
- **Skeleton before read** — `get_file_skeleton(filePath)` then read specific lines
- **Usages before edit** — `find_usages(symbolName)` before modifying any symbol
- **Images via describe** — `describe_image(filePath, systemPrompt?)` — never read raw bytes

If no results, run `opencode-rag index`.

## Architecture

Entry points: `src/index.ts` (library), `src/plugin-entry.ts` (OpenCode plugin), `src/cli.ts` (CLI), `src/tui.ts` (TUI), `src/web/server.ts` (Web UI).

Core modules: `src/core/` (config, interfaces, manifest), `src/chunker/` (AST chunking), `src/embedder/` (Ollama/OpenAI/Cohere), `src/describer/` (LLM descriptions), `src/retriever/` (vector + keyword hybrid), `src/vectorstore/` (LanceDB), `src/opencode/` (plugin integration).

Full architecture: [doc/architecture.md](doc/architecture.md).

## Known Gotchas

- **npm install**: use `--legacy-peer-deps` (LanceDB peer dep conflicts)
- **LanceDB types**: cast through `unknown` — `rows as unknown as Record<string, unknown>[]`
- **LanceDB index metric**: the IVF index on `embedding` must use `distanceType: "cosine"` to match `searchInternal` (default is `l2`, which makes every query log "Requested metric Cosine is incompatible" and fall back to brute-force). `LanceDbStore.ensureCosineIndex()` self-heals stale L2 indexes on first search. When replacing an index, use a single `createIndex(..., replace: true, waitTimeoutSeconds)` — a `dropIndex` + `createIndex` sequence races and fails with "Retryable commit conflict".
- **LanceDB "partition N is empty, skipping" warnings**: benign once per index build (IVF KMeans on duplicate/degenerate vectors). Constant spam = retrain churn from a store whose index commits never register (one new `_indices/<uuid>` dir per attempt). `repairIndexMetricOnce` guards: counts only index-version dirs WITH files (empty husks from version pruning never trip it), verifies post-createIndex registration via `indexStats`, gives up after 3 failed attempts per process, and skips IVF creation below `MIN_ROWS_FOR_IVF_INDEX` (4096 rows — Lance's own floor for a "meaningful index"; brute-force is optimal there); `optimize()` sweeps empty husk dirs, and rebuilds pass `optimize({ skipIndex: true })` to temp-store mid-run optimizes so the index is built once at the end. Fix for a truly non-converging store: delete rag_db + reindex. Note: u64-near `_versions/*.manifest` names are NORMAL (counter starts at u64::MAX-1 and decrements) — not corruption.
- **LanceDB "KMeans: more than 10% of clusters are empty" warning** (e.g. `2 of 16 ... 2507 < 4096`): benign and one-time when printed during the single IVF build at the end of a first-time index. Emitted by Lance's native Rust logger (`lance_index::vector::kmeans`), NOT by OpenCodeRAG's logger, so it can't be routed to the plugin log; it repeats once per KMeans training pass. See doc/troubleshooting.md.
- **LanceDB dimension integrity**: LanceDB silently zero-pads/truncates embeddings whose length differs from the table's `FixedSizeList` column — mismatched writes SUCCEED and produce unreachable rows, while queries fail with `No vector column found to match with the query vector dimension: N`. `LanceDbStore` now caches the schema dimension (`getVectorDimension()`) and throws `DimensionMismatchError` on mismatched writes; `searchWithFilter` compares against the *table* dimension, not the handle's. `runIndexPass` treats a schema mismatch as a rebuild condition (clears manifest, atomic `rag_db_tmp` rebuild at the configured dimension — also when the store is empty). `resolveRagContext` dimension precedence: `embedding.vectorDimension` > probe > existing store schema > 384. Full rebuild swaps preserve `quirks.jsonl`/`.desc-cache.json`/`runtime-overrides.json`/`eval-sessions/` (they live in the store dir and are NOT re-derivable).
- **Embedding preflight**: `runIndexPass` probes the embedder before chunking/description and aborts (`embeddingUnavailable: true`, CLI exit 1) when the provider is down or its dimension disagrees with `options.dimension` — prevents a 6-minute "successful" pass that stores 0 chunks.
- **`indexing.embedDescriptions`** (default `true`): set `false` for code-specialized embedders (e.g. jina-code-embeddings) — descriptions are still generated/stored for display but excluded from the embedded text.
- **`describe_image` model split**: on-demand calls (plugin tool, MCP server, `opencode-rag describe-image`) resolve `imageDescription.onDemand` overrides on top of the indexing config via `resolveOnDemandImageConfig` (src/chunker/image.ts); indexing always uses the base `imageDescription.*` and ignores `onDemand`, so it needs no re-index. `imageDescription.enabled` stays the master gate. Health checks emit `image_description_on_demand` when the effective provider/model differs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MrDoe/OpenCodeRAG](https://github.com/MrDoe/OpenCodeRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
