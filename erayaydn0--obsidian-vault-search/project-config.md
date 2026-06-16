---
trigger: always_on
description: Operational guide for AI agents working in this repository. Read this fully before making changes.
---

# AGENTS.md — VaultSearch

Operational guide for AI agents working in this repository. Read this fully before making changes.

---

## 1. What This Project Is

**VaultSearch** is an Obsidian community plugin that provides **local-first hybrid search** across a user's vault. Everything runs on-device — no cloud, no telemetry, no external network calls except the one-time embedding model download.

- **License:** MIT
- **Platform:** Obsidian (Electron) — `isDesktopOnly: true`
- **Plugin id:** `vault-search`
- **Status:** Early development. The indexer, storage, search engine, and embedding Web Worker pipeline are in place. **There is no MCP server** in this repository.

### Goals

1. Fast hybrid search combining BM25, vector similarity, and fuzzy title matching.
2. Zero configuration: works out of the box on any vault.
3. Multilingual semantic search via a quantized sentence-transformers model.
4. No native dependencies, no cloud calls, no telemetry.

---

## 2. Tech Stack

| Concern | Choice | Notes |
|---|---|---|
| Package manager / runner | **Bun ≥ 1.3.0** | Toolchain only. Lockfile: `bun.lock` (committed). |
| Language | TypeScript, target **ES2020** | `strict` mode. **`any` is forbidden.** |
| UI | **Plain Obsidian API** (no Svelte, no React) | `esbuild-svelte` is wired in `esbuild.config.mjs` for future use, but `src/ui/` is currently 100% `.ts`. Do not introduce `.svelte` files without prior agreement. |
| Bundler | esbuild (`esbuild.config.mjs`) | Emits `main.js` + `worker.js` at the plugin root and copies `ort-wasm-simd-threaded.wasm` (see §6). |
| Test runner | **`bun test`** | Tests import from `bun:test`. |
| SQLite | **`sql.js`** (pure WASM) | NOT `wa-sqlite`, NOT `better-sqlite3`. Native addons do not load in the Obsidian sandbox. |
| Embeddings | **`@huggingface/transformers` v4.x** (ONNX/WASM) | NOT `@xenova/transformers`. |
| Default model | `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` | 384-dim, ~47 MB quantized, 50+ languages. |
| Vector search | Pure-JS brute-force cosine similarity | No `sqlite-vec` (native C extension). |

---

## 3. Repository Layout

The plugin lives at `.obsidian/plugins/obsidian-vault-search/` inside a host vault, so you can develop directly in-place and reload Obsidian.

```
.
├── src/
│   ├── main.ts                  # Plugin entry: lifecycle, commands, ribbon, vault events
│   ├── constants.ts             # PLUGIN_NAME, view types, default tunables
│   ├── types.ts                 # Public types + DEFAULT_SETTINGS
│   ├── sqlJsBundled.ts          # sql.js bootstrap (WASM bytes inlined)
│   ├── sqlJsRuntime.ts          # sql.js runtime initialization
│   ├── core/
│   │   ├── VaultIndexer.ts      # Orchestrates initial scan + incremental file events
│   │   ├── EmbeddingEngine.ts   # Web Worker + hash fallback; Float32Array embeddings
│   │   ├── SearchEngine.ts      # Hybrid search: BM25 + vector + title, RRF fusion
│   │   ├── FileParser.ts        # Markdown → chunks (heading-aware, token-bounded)
│   │   └── SQLiteStore/
│   │       ├── index.ts         # Public barrel
│   │       ├── SQLiteStore.ts   # Main store class
│   │       ├── schema.ts        # Tables, FTS5 virtual table, triggers
│   │       ├── writeOps.ts      # Insert/update/delete files & chunks
│   │       ├── searchOps.ts     # BM25 query, candidate retrieval
│   │       ├── statsOps.ts      # Index stats
│   │       ├── scoring.ts       # Score helpers
│   │       ├── cacheManager.ts  # In-memory embedding matrix cache
│   │       ├── helpers.ts
│   │       └── storeTypes.ts
│   ├── workers/
│   │   └── embeddingWorker.ts   # Bundled separately → worker.js (ONNX/transformers)
│   ├── ui/
│   │   ├── SearchModal.ts       # Cmd/Ctrl+Shift+F search modal
│   │   ├── SidebarView.ts       # Right-pane "related notes" view
│   │   └── SettingsTab.ts       # Settings panel
│   ├── utils/
│   │   ├── jaroWinkler.ts       # Fuzzy title scoring
│   │   ├── snippetExtractor.ts  # Highlight excerpts from chunk content
│   │   └── tokenCounter.ts      # Approximate token counting for chunking
│   └── types/
│       └── wasm.d.ts            # Ambient declarations for .wasm imports
├── test/
│   ├── unit/                    # Pure unit tests for parsers, scoring, store
│   ├── integration/             # End-to-end indexer/search behavior
│   └── helpers/
│       ├── testHarness.ts       # Bun test setup
│       └── VaultTestContainer.ts # In-memory vault fixture
├── scripts/
│   └── test-docker.ts           # Reproducible Docker-based test runner (see §7)
├── esbuild.config.mjs
├── eslint.config.mjs
├── manifest.json                # Obsidian plugin manifest
├── package.json
├── bun.lock                     # Committed
├── tsconfig.json
├── versions.json                # Min Obsidian app version per plugin release
├── styles.css                   # Plugin styles (loaded by Obsidian)
├── Dockerfile.test
├── CLAUDE.md                    # → defers to this file
└── AGENTS.md                    # ← you are here
```

**Build outputs** at the plugin root (generated by `bun run dev` / `bun run build`; listed in `.gitignore` for source-only workflows):

- `main.js` — main plugin bundle (sql.js WASM bytes inlined).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erayaydn0/obsidian-vault-search](https://github.com/erayaydn0/obsidian-vault-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
