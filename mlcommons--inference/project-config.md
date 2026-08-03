---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a RAG (Retrieval-Augmented Generation) benchmark system for evaluating multi-hop question answering using Wikipedia documents from the FRAMES dataset. The system supports multiple retrieval methods (BM25, vector search), reranking, and multi-shot iterative retrieval with query decomposition.

## Architecture

The codebase follows a modular pipeline architecture:

1. **Document Ingestion** → **Passage Chunking** → **Vector/BM25 Indexing**
2. **Query** → **Retrieval** → **Optional Reranking** → **LLM Answer Generation** → **Evaluation**

### Core Components

- **`retrieve/` module**: Defines abstract `RagDB` base class with two implementations:
  - `VectorDB`: Dense vector search using FAISS (flat, HNSW, or IVF indexing)
  - `BM25DB`: Sparse lexical search using bm25s library
  - Both support reranking with ColBERTv2 or similar models
  
- **Retrieval Scripts**:
  - `single_shot_retrieval.py`: Single-step retrieval and evaluation
  - `multi_shot_retrieval.py`: Multi-hop retrieval with LLM-based query decomposition (iterative retrieval with query rewriting)
  - `oracle_single_shot.py`: Upper-bound evaluation using ground truth documents

- **Parameter Management**: `params.py` centralizes all CLI parameters with Optuna optimization metadata

- **Utilities**:
  - `download_docs.py`: Downloads Wikipedia pages from FRAMES dataset URLs
  - `read_docs.py`: Extracts text and chunks documents into passages (uses `text_splitter.py`)
  - `evaluate.py`: LLM judge-based evaluation of generated answers
  - `utils.py`: Common helpers (device config, LLM setup, seeding)

## Common Development Workflows

### Initial Setup

```bash
# Install dependencies (requires Ubuntu-based environment)
./setup.sh

# Download Wikipedia documents from FRAMES dataset
python3 download_docs.py --output_dir doc_html --format html --processes 30

# Build vector database (run once)
bash scripts/run_ingestion.sh
```

The setup script will:
1. Extract passages from HTML documents → `passages/doc_html_len2048_ov32_word.json`
2. Build FAISS HNSW index → `vector_html_hnsw_len2048_ov32_word.db`

### Running Experiments

**Single-shot retrieval:**
```bash
# Run evaluation on existing database
python3 single_shot_retrieval.py \
    --db vector_html_hnsw_len2048_ov32_word.db \
    --retrieval_method vector \
    --eval 100 \
    --generate-answer

# Compare BM25 vs Vector
python3 single_shot_retrieval.py --db bm25.db --retrieval_method bm25 --eval
```

**Multi-shot retrieval with query decomposition:**
```bash
# Run multi-shot experiment (requires LLM server on port 8123)
bash scripts/run_multi_shot.sh 50  # Evaluate 50 queries
bash scripts/run_multi_shot.sh all # Full evaluation
```

**Oracle upper bound (using ground truth docs):**
```bash
python3 oracle_single_shot.py \
    --dataset data/frames_dataset.tsv \
    --wiki-articles-dir wiki_articles \
    --batch-size 16
```

### LLM Server Setup

The system expects a vLLM-compatible OpenAI API server:

```bash
# Start vLLM server (example from scripts/start_vllm_server.sh)
python3 -m vllm.entrypoints.openai.api_server \
    --model /model/gpt-oss-20b-mxfp4 \
    --dtype bfloat16 \
    --host 0.0.0.0 \
    --port 8123 \
    --gpu-memory-util=0.95 \
    --enable-prefix-caching \
    --max-model-len=131072
```

Default service URL: `http://127.0.0.1:8123/v1/chat/completions`

### Evaluation

Evaluation uses an LLM judge to score answers:

```bash
# Score results from any experiment
python3 evaluate.py result_single_shot.json
python3 evaluate.py result_multi_shot.json
python3 evaluate.py oracle_checkpoint.pkl  # For oracle results
```

## Key Parameters (via params.py)

All parameters are centralized in `params.py` with CLI definitions. Key categories:

**Retrieval Method:**
- `--retrieval_method {bm25,vector}`: Choose retrieval backend
- `--vector_index_method {flat,hnsw,ivf}`: FAISS index type (default: hnsw)
- `--bm25_k1`, `--bm25_b`, `--bm25_stemmer`: BM25 tuning parameters

**Retrieval Strategy:**
- `--retrieval_strategy {fixed_k,top_p,relative}`: How many docs to retrieve
- `--top_k_retriever N`: Number of docs to retrieve (default: 10)
- `--top_k_reranking N`: Number of docs after reranking (default: 10)

**Device & Performance:**
- `--device {auto,xpu,cuda,hpu,cpu}`: Hardware accelerator
- `--num_embedding_devices N`: Parallel embedding generation across devices
- `--benchmark`: Enable performance monitoring

**Multi-shot specific (multi_shot_retrieval.py):**
- `--max-iterations N`: Max retrieval rounds (default: 5)
- `--max-sub-queries N`: Sub-queries per iteration (default: 3)

**Oracle specific (oracle_single_shot.py):**
- `--batch-size N`: Batch requests to LLM (default: 1)
- `--timeout N`: Request timeout in seconds (default: 2400)
- `--enable-thinking`: Use chain-of-thought reasoning

## Hardware Support

The system supports multiple accelerators via `--device`:
- **XPU** (Intel GPUs): Primary development target
- **CUDA** (NVIDIA GPUs)
- **HPU** (Habana Gaudi): Embeddings/reranking fall back to CPU due to compatibility
- **CPU**: Fallback option


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlcommons/inference](https://github.com/mlcommons/inference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
