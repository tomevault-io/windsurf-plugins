---
trigger: always_on
description: A dynamic context and memory storage system for coding agents. Provides native hybrid retrieval (vector + full-text + app-side re-ranking) backed by Meilisearch with local or remote embeddings. Exposes two interfaces: CLI and REST API (dashboard).
---

# mairu

A dynamic context and memory storage system for coding agents. Provides native hybrid retrieval (vector + full-text + app-side re-ranking) backed by Meilisearch with local or remote embeddings. Exposes two interfaces: CLI and REST API (dashboard).

## Tech Stack

- **Runtime:** Go 1.25+
- **Database:** Meilisearch 1.12+ (Docker)
- **Search:** Native hybrid — vector cosine similarity + full-text + app-side re-ranking (importance, recency decay)
- **Embeddings:** fastembed (local, `fast-all-MiniLM-L6-v2`, 384 dims) or OpenAI-compatible API
- **Frontend:** Svelte 5 + Vite
- **Testing:** Go testing (`go test`)
- **Linting:** `golangci-lint` (fallback: `go vet`)

## Setup

```bash
docker compose up -d    # start Meilisearch
bun install
bun --cwd mairu/ui install
cp .env.example .env    # fill in MEILI_URL, GEMINI_API_KEY, EMBEDDING_PROVIDER
make setup              # create Meilisearch indexes (destructive — drops and recreates)
```

## Commands

| Command | Description |
|---|---|
| `docker compose up -d` | Start Meilisearch container |
| `docker compose down` | Stop Meilisearch container |
| `make build` | Compile Go binary to `bin/mairu` |
| `make test` | Run Go tests |
| `make lint` | Run Go lint checks |
| `make clean` | Remove `mairu/bin/` |
| `make setup` | Init/reset Meilisearch indexes |
| `make dashboard` | Start context server (API) + Svelte dev UI |
| `bun run dashboard:dev` | Start Svelte dev server on port 5173 |
| `bun run dashboard:build` | Build Svelte UI |

### Evaluation

```bash
./mairu/bin/mairu eval:retrieval --dataset ./llmeval/sample_dataset.json --topK 5 --verbose true
./mairu/bin/mairu eval:retrieval --dataset ./llmeval/sample_dataset.json --topK 5 --fail-below-mrr 0.8 --fail-below-recall 0.75
```

## Architecture

### Data Types

- **Memories** — facts with category, owner, importance (1–10)
- **Skills** — capability name + description pairs
- **Context Nodes** — hierarchical tree nodes with abstract/overview/content levels, addressed by URI

### Retrieval Pipeline

Meilisearch handles vector + full-text search natively; app-side re-ranking applies recency decay and importance boosting:

1. **Vector search** — dense vector cosine similarity on embeddings
2. **Full-text** — Meilisearch built-in keyword search
3. **App-side re-ranking** — exponential recency decay + importance score boost
4. Results from both retrievers are merged and re-ranked before returning

Weights (vector, keyword, recency, importance) are defined in `mairu/internal/contextsrv/search_rerank.go`.

### Meilisearch Indexes

| Index | Key Fields |
|---|---|
| `contextfs_skills` | name (text), description (text), embedding (dense_vector), project (keyword) |
| `contextfs_memories` | content (text), category/owner (keyword), importance (integer), embedding (dense_vector) |
| `contextfs_context_nodes` | name/abstract/overview/content (text), uri/parent_uri (keyword), ancestors (keyword[]), embedding (dense_vector) |

### Search Features

| Feature | Description | Controlled by |
|---|---|---|
| **Vector search** | Dense cosine similarity on embeddings | `weights.vector` |
| **Full-text** | Meilisearch keyword search | `weights.keyword` |
| **Synonyms** | Custom synonym expansion (e.g., "k8s" → "kubernetes") | `SYNONYMS` env var |
| **Importance boost** | App-side boost on importance (1-10) | `weights.importance` |
| **Recency decay** | Exponential decay on created_at | `weights.recency`, `RECENCY_SCALE`, `RECENCY_DECAY` |
| **Min score cutoff** | Hard threshold to drop low-confidence results | `--minScore` |
| **Highlights** | Returns `<mark>`-tagged snippets showing matched terms | `--highlight` |
| **Field boosts** | Per-search field weight overrides | `fieldBoosts` option (API only) |

### Key Modules

| File | Role |
|---|---|
| `mairu/internal/contextsrv/meili.go` | Meilisearch integration, hybrid retrieval orchestration |
| `mairu/internal/contextsrv/service.go` | High-level API used by CLI |
| `mairu/internal/llm/openai_embedder.go` | OpenAI-compatible embedding calls |
| `mairu/internal/contextsrv/search_rerank.go` | Hybrid score blending and reranking weights |
| `mairu/internal/llm/router.go` | LLM-powered deduplication (CREATE / UPDATE / SKIP) |
| `mairu/internal/llm/ingestor.go` | Free-form text → structured context nodes |
| `mairu/internal/contextsrv/service_vibe.go` | LLM-driven free-text query planning and mutation planning |
| `mairu/cmd/mairu/main.go` | CLI entry point |
| `mairu/internal/web/server.go` | REST API for dashboard |
| `mairu/internal/eval/evaluate.go` | Evaluation harness entry point |
| `mairu/internal/daemon/daemon.go` | File watcher daemon: parallel processing, persistent cache, NL content assembly |
| `mairu/internal/ast/language_describer.go` | Pluggable interface for language-specific AST extraction + shared types/utilities |
| `mairu/internal/ast/typescript_describer.go` | TypeScript/JS implementation of LanguageDescriber (tree-sitter based) |
| `mairu/internal/ast/nl_describer.go` | AST-to-English engine: converts function bodies to numbered NL descriptions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enekos/mairu](https://github.com/enekos/mairu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
