---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Project Is

MODA is an open-source, end-to-end fashion search project with three parallel workstreams:

1. **H&M full-pipeline search benchmark** (`benchmark/` + `scripts/`) — measures every retrieval component (BM25, SPLADE, dense/CLIP, hybrid fusion, NER, cross-encoder reranking) against 253,685 purchase-grounded H&M queries × 105,542 products.
2. **Beat-FashionSigLIP track** (`scripts/v4/`) — fine-tunes Marqo-FashionSigLIP via GCL with pattern-targeted + synthetic data, with the explicit goal of exceeding FashionSigLIP's published MRR numbers on Marqo's own 7-dataset benchmark.
3. **HuggingFace model publishing** (`hf_repos/`) — five distilled fashion embedding models, each self-contained with its own `inference.py`.

---

## Environment Setup

```bash
# Python 3.10+, virtualenv at .venv/
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# For GPU-only training (A100 runs)
pip install -r scripts/requirements_gpu.txt

# Start OpenSearch (required for any BM25 or SPLADE pipeline)
docker compose up -d
# OR one-liner without compose:
docker run -d -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e DISABLE_SECURITY_PLUGIN=true \
  opensearchproject/opensearch:2.11.0

# Sanity check the full environment
python scripts/verify_setup.py
```

---

## Track 1 — H&M Search Pipeline

### Architecture

```
Query
  ├─► SPLADE (naver/splade-cocondenser-ensembledistil)  ─┐
  ├─► Dense  (Marqo-FashionCLIP → FAISS)                ─┼─► RRF Fusion → CE Reranker → Top-10
  └─► BM25   (OpenSearch 2.11, optional)                ─┘
      [optional: GLiNER NER attribute boosts]
```

Best config: `SPLADE ⊕ FT-FashionCLIP (0.3/0.7) + LLM-trained CE` → nDCG@10 = 0.1063

### Data Preparation

```bash
# Download H&M data (253K queries → data/raw/hnm_real/)
python scripts/build_hnm_benchmark.py

# Index articles in OpenSearch
python benchmark/index_hnm_opensearch.py

# Embed 105K articles with FashionCLIP → FAISS index
python benchmark/embed_hnm.py --model fashion-clip

# Embed images for multimodal pipeline
python benchmark/embed_hnm_images.py
```

### Evaluation

```bash
# Full 253K overnight run (staged, checkpointed — can resume)
python benchmark/eval_full_253k.py --stages all
python benchmark/eval_full_253k.py --stages 1    # BM25 + FAISS only
python benchmark/eval_full_253k.py --stages 3    # CE reranking (8.5 hrs)

# Faster 10K sample (for iteration)
python benchmark/eval_hybrid.py
python benchmark/eval_full_pipeline.py

# SPLADE evaluation
python -m benchmark.eval_splade_pipeline          # test split (22K queries)
python -m benchmark.eval_full_253k_splade         # full 253K

# Multimodal pipeline
python benchmark/eval_multimodal_pipeline.py
bash scripts/run_phase4g_multimodal_eval.sh

# Specific ablations
python benchmark/eval_gliner2_ablation.py         # GLiNER v1 vs GLiNER2
python benchmark/eval_lookbench_baseline.py       # LookBench zero-shot baselines
```

### Training

```bash
# Fine-tune cross-encoder with LLM labels
python benchmark/train_cross_encoder.py

# Fine-tune bi-encoder (FashionCLIP) with hard negatives
python benchmark/train_biencoder.py

# Three-tower architecture training
python benchmark/train_three_tower.py --quick    # smoke test
python benchmark/train_three_tower.py            # full run
```

### Key Modules

| File | Role |
|------|------|
| `benchmark/metrics.py` | All IR metrics: nDCG, MRR, Recall, AP — used by all eval scripts |
| `benchmark/models.py` | `MODEL_REGISTRY` + loaders for `open_clip` and `sentence-transformers` |
| `benchmark/query_expansion.py` | `FashionNER` (GLiNER) + `SynonymExpander`; `build_boosted_query()` → OpenSearch |
| `benchmark/splade_retriever.py` | `SpladeRetriever` — encode articles/queries into sparse vectors, retrieve via dot product |
| `benchmark/article_text.py` | `build_article_text()` — canonical text representation of H&M articles for indexing |
| `benchmark/_faiss_flat_worker.py` | FAISS in subprocess to avoid MPS/BLAS conflicts on Apple Silicon |

### Evaluation Splits

- **Phase 2 (zero-shot configs):** all 253,685 queries — nothing was trained on this data
- **Phase 3+ (trained models):** 22,855-query held-out test split — disjoint from training to prevent leakage
- Data leakage checks: `benchmark/data_leakage_check_extended.py`

### Output Locations

- `results/full/full_ablation.json` — 253K final ablation results
- `results/real/` — 10K sample and test-split results
- `results/lookbench/` — LookBench per-subset JSON files

Long-running stages checkpoint to disk automatically. Re-running `eval_full_253k.py` with the same `--stages` flag resumes without repeating completed work. Run `caffeinate -i &` before overnight jobs on macOS to prevent sleep.

---

## Track 2 — Beat-FashionSigLIP

**Goal:** exceed Marqo-FashionSigLIP's published MRR numbers on Marqo's own 7-dataset benchmark using GCL fine-tuning with pattern-targeted + synthetic training data.

### Targets (FashionSigLIP zero-shot baseline, MRR)

From `results/v4_gcl/baseline_v4/full_results.json` — these are the numbers we have to beat:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hopit-ai/Moda](https://github.com/hopit-ai/Moda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
