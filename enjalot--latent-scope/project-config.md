---
trigger: always_on
description: Guidance for AI agents (Claude Code and others) working on this codebase. If you
---

# CLAUDE.md — Latent Scope

Guidance for AI agents (Claude Code and others) working on this codebase. If you
are an agent asked to "run the pipeline" or "show me the results", jump to
[Agent quickstart](#agent-quickstart-run-it-for-the-user).

---

## What Latent Scope does

Latent Scope is a Python + React tool for embedding, projecting, clustering,
labeling, and exploring datasets through the lens of their latent space. The
backend is a Flask server; the frontend is a React/Vite SPA. Pipeline steps run
as CLI subprocesses (`ls-embed`, `ls-umap`, …) that can be driven from the web
UI or scripted directly.

```
ingest → embed → umap → cluster → label → scope → (sprite atlas) → explore
```

### Capabilities (what an agent can offer a user)

- **Text and image datasets.** Ingest CSV/Parquet/JSON/JSONL/XLSX or a pandas
  DataFrame. Image columns (HF `{bytes,path}` dicts, raw bytes, or URLs) are
  auto-detected and embeddable (issue #87).
- **Dense embeddings** from many providers (sentence-transformers/HF,
  OpenAI, Cohere, Voyage, Mistral, Together, and any OpenAI-compatible endpoint).
- **ColBERT late-interaction (multi-vector) embeddings** via `pylate` — per-token
  vectors stored fp16, searched with MaxSim (issue #64). See
  `examples/colbert_quickstart/`.
- **LanceDB vector storage** (replaced HDF5). Embeddings live in a per-dataset
  LanceDB table; old HDF5 embeddings are migrated on demand.
- **UMAP projection + HDBSCAN/EVoC clustering**, with LLM cluster labeling.
- **Explore UI**: GPU scatterplot with hover/select, density heatmap, cluster
  outlines, similarity + late-interaction search, filtering, tagging.
- **Image map (sprite atlas)**: for image datasets the map is a continuous
  level-of-detail — heatmap when zoomed out, a tiled representative-image
  pyramid as you zoom in, then individual points on top for hovering. Built by
  the optional post-scope `ls-sprite-atlas` step (issue #24).

---

## Agent quickstart: run it for the user

The data directory is set by the `LATENT_SCOPE_DATA` environment variable. On a
dev checkout, use `uv run` (it resolves `.venv` automatically).

**Run the full pipeline on a CSV (text):**
```bash
export LATENT_SCOPE_DATA=~/latent-scope-data        # or wherever data should live
uv run ls-ingest mydata --path /path/to/data.csv --text_column text
uv run ls-embed   mydata text transformers-BAAI___bge-small-en-v1.5
uv run ls-umap    mydata embedding-001 25 0.1
uv run ls-cluster mydata umap-001 5 3 0.0 --method hdbscan
uv run ls-scope   mydata embedding-001 umap-001 cluster-001 default "My scope" "description"
```
`cluster` auto-writes a `…-labels-default` parquet, so `ls-scope … default …`
works **without** an LLM. For nicer labels run `ls-label` with a chat model first
and pass that labels id instead of `default`.

**Show the user the results (single command, recommended):**
```bash
uv run ls-serve $LATENT_SCOPE_DATA            # serves API + built web UI at http://localhost:5001
```
Open `http://localhost:5001`, pick the dataset, open the scope. This serves the
**pre-built** web assets — no Node required. Use this when the user just wants to
look at results.

**Live frontend dev (two processes, only when changing React):**
```bash
uv run ls-serve $LATENT_SCOPE_DATA            # API on :5001
cd web && npm install && npm run dev          # Vite dev server on :5173 -> proxies to :5001
```

**End-to-end example to copy from:** `examples/colbert_quickstart/run.sh` builds
a tiny topical dataset, embeds it with a small ColBERT model on CPU, runs the
whole pipeline, and verifies late-interaction search. Run it with
`bash examples/colbert_quickstart/run.sh`.

> CPU-only / shared-GPU machines: prefix commands with `CUDA_VISIBLE_DEVICES=`
> to force CPU. Small HF models (e.g. `bge-small-en-v1.5`,
> `answerai-colbert-small-v1`) embed fine on CPU.

See [`docs/data-importing.md`](docs/data-importing.md) for the full set of input
formats, column detection rules, and how to import precomputed embeddings.

---

## Repository Layout

```
latentscope/                 # Python package
  __init__.py                # Public API (lazy imports for heavy deps)
  models/
    __init__.py              # get_embedding_model(), get_chat_model()
    providers/               # transformers, openai, cohere, voyage, late_interaction (ColBERT), ...
    embedding_models.json    # registry (dense + colbert-* late-interaction models)
    chat_models.json
  scripts/                   # Pipeline step implementations (each has a CLI + a function)
    ingest.py, embed.py, umapper.py, cluster.py, label_clusters.py, scope.py,
    sprites.py,              # per-row image sprites (legacy serving)
    sprite_atlas.py          # tiled representative-image atlas pyramid (image map)
  server/                    # Flask application
    app.py                   # create_app() factory; LRU caches
    jobs.py                  # subprocess job runner + routes (list-based, never shell=True)
    datasets.py              # dataset/scope/atlas routes
    search.py                # nn search + nn_late_interaction (MaxSim)
    tags.py, bulk.py, admin.py, models.py, estimate.py
  util/
    configuration.py         # LATENT_SCOPE_DATA, API keys, dotenv helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enjalot/latent-scope](https://github.com/enjalot/latent-scope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
