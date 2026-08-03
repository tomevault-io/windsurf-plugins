---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repo.
---

# CLAUDE.md — multimodal-rag-system

Guidance for AI agents (and humans) working in this repo.

## What this project is

Two layers:

1. **The app** — a production-style RAG Q&A system for a personal website:
   Streamlit UI (`frontend/personal_app.py`) → FastAPI (`src/api/personal_api.py`)
   → pluggable retrievers (`src/rag/`) over a personal knowledge base. Mock mode
   runs with zero API cost; live demo on Streamlit Cloud. `python run.py` starts both.

2. **The evidence engagement (active work)** — an *eval-first* effort to back specific
   résumé claims with **real, reproducible numbers** on a public benchmark, and to
   correct any claim that isn't true. **Integrity rule: never fabricate to hit a
   target; report the real number.** Benchmark = **Flickr30k caption→image retrieval**.

## Locked technical decisions

- **Vector store**: self-hosted **Qdrant** is the real backing store; `src/rag/vector_store.py`
  keeps Pinecone/Chroma pluggable via `VECTOR_BACKEND` env. Résumé says Qdrant.
- **Dataset**: Flickr30k (~31K images × 5 captions). "50K+ items" = the ~62K ingested vectors.
- **Retrieval**: hybrid **dense (CLIP) + sparse (BM25)** fused with **RRF**, then a
  **cross-encoder reranker** (`bge-reranker-v2-m3`, local, text-only).
- **Eval protocol**: standard Flickr30k. Report **both** the standard 1K-gallery
  protocol (headline) **and** the full ~31K corpus, always with 95% CIs.
- **Hardware**: Mac mini M4, 16GB. Load models **on demand**; never CLIP + reranker +
  a 7B LLM at once (CLIP ViT-B/32 ~600MB, bge-reranker ~2.2GB).

## Environment & key commands

Eval/retrieval work uses a dedicated venv (system python is 3.9; repo targets 3.11-3.12):

```bash
source .venv-eval/bin/activate          # uv venv, py3.12; deps in requirements_eval.txt
                                        # NOTE: CLIP via PyPI `clip-anytorch` (not the git source);
                                        # datasets pinned to 2.21 (v4 dropped script loaders)
python -m eval.datasets.prepare_flickr30k    # build gallery + eval_queries{,_5k}.jsonl (gitignored)
python -m scripts.ingest                     # CLIP-encode full Flickr30k → Qdrant (~62K vectors)
python -m eval.evaluate --retrievers clip --queries 5k --tag headline   # standard-protocol R@5 + CI
python -m eval.ablation_rerank --rerank-n 50                            # dense vs +RRF vs +rerank
python -m eval.faithfulness_check                                       # vector-DB == numpy proof
pytest tests/test_eval_metrics.py tests/test_fusion.py -q               # metric/fusion unit tests

# Phase 3b — stronger dense backbone (OpenCLIP ViT-H/14, laion2b, 1024-d). Needs:
#   uv pip install open_clip_torch>=2.24.0
python -m eval.evaluate --retrievers openclip --queries 5k --tag phase3b_headline   # ViT-H/14 R@5 (numpy, 1K gallery — the headline)
python -m scripts.ingest --backbone open_clip                          # re-ingest 62K vectors → flickr30k_openclip collection
python -m eval.evaluate --retrievers qdrant_openclip_full --queries 1k --tag phase3b_scale  # ViT-H/14 31K-corpus number (slow)
```

Perf note: **Qdrant local mode = exact full-scan** (~0.28s/query over 62K), so 5K-query
Qdrant runs take ~20 min. For the 1K headline use the `clip` numpy adapter (fast; proven
identical to Qdrant via `faithfulness.json`). Eval adapters batch-encode queries
(`encode_queries`) to avoid per-query CLIP overhead.

## New code added by this engagement

- `eval/metrics.py` — Recall@k, MRR, nDCG@10 + 95% CIs (Wald); unit-tested.
- `eval/datasets/prepare_flickr30k.py` — 1K gallery; `eval_queries.jsonl` (held-out, proxy-safe)
  and `eval_queries_5k.jsonl` (standard 5000-caption protocol).
- `eval/adapters.py` — uniform `retrieve_ids` over clip/openclip/mock/dense/qdrant{,_openclip}{,_full}.
- `eval/evaluate.py` — runner; `--queries 1k|5k`, `--tag`, CI columns.
- `eval/ablation_rerank.py` — dense vs +RRF vs +rerank ablation.
- `eval/faithfulness_check.py` — isolated proof the vector DB preserves rankings.
- `src/rag/vector_store.py` — pluggable Qdrant/Pinecone VectorStore (embedder-agnostic).
- `scripts/ingest.py` — batched CLIP ingest of full Flickr30k.
- `src/rag/sparse_retriever.py` (BM25), `src/rag/fusion.py` (RRF), `src/rag/reranker.py` (cross-encoder).
- `src/rag/openclip_retriever.py` — **Phase 3b** OpenCLIP ViT-H/14 (laion2b, 1024-d) retriever; mirrors
  `CLIPRetriever`'s interface, embed-dim probed (never hard-coded). `eval/adapters.py` adds `openclip`
  (numpy fast path → headline) and `qdrant_openclip{,_full}` (vector-DB); `scripts/ingest.py --backbone
  open_clip` writes 1024-d vectors to a SEPARATE `flickr30k_openclip` collection.
- `src/rag/image_search_service.py` — **Phase 4** lazy + gated bridge from the measured OpenCLIP
  gallery to the served API; import-safe under `requirements_simple` (heavy deps loaded only on first
  use), returns 503 when no gallery cache (keeps the mock deployment working). Backs the new
  `POST /search/text` (caption→image) and `POST /search/image` (reverse image) FastAPI endpoints.
- `eval/qualitative_demo.py` — **Phase 2** writes `eval/results/phase2_qualitative.md`: text→image and
  image→image retrieved examples (by image_id + caption) — the qualitative companion to R@5=0.942.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhengbrody/multimodal-rag-system](https://github.com/zhengbrody/multimodal-rag-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
