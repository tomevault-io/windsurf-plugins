---
trigger: always_on
description: This file encodes the non-negotiable rules for working on the Findegil codebase.
---

# Findegil — AI Agent & Contributor Directives (CLAUDE.md)

This file encodes the non-negotiable rules for working on the Findegil codebase.
Whether you are a human contributor or an AI agent (Claude, Cursor, Copilot, etc.), read this at the start of every session.

Findegil's primary value proposition is **extreme efficiency and operational simplicity**. We explicitly reject "Big Data" complexity for e-commerce catalogs that fit comfortably in RAM. Every rule here defends that premise. If a task seems to require violating a rule, the answer is **never** to violate it silently. Write an ADR (Architecture Decision Record) and surface the proposal to the human reviewer.

---

## 1. The Stack & Forbidden Technology

Hard dependencies. Substitutions require an ADR.

- **Language:** Python 3.13. Pinned in `.python-version`.
- **Package manager:** `uv`. Never run `pip`, `poetry`, `conda`. All commands go through `uv run …` or `uv sync`.
- **HTTP:** FastAPI + uvicorn (ASGI).
- **Lexical search:** `tantivy-py` (Rust, embedded, in-process).
- **Embeddings model:** `intfloat/multilingual-e5-small`, exported to ONNX INT8.
- **Inference:** `onnxruntime` (CPU execution provider only).
- **Learning-to-rank:** `catboost` with `YetiRank` loss, native `.cbm` format.
- **Numerical:** `numpy`, `scikit-learn` (training only).
- **Lint/format/types:** `ruff`, `ty` (Astral's type checker).
- **Test:** `pytest` with `pytest-benchmark` for latency tests.

**Forbidden without an approved ADR:** Elasticsearch / OpenSearch, any vector database (Pinecone, Weaviate, Qdrant, pgvector, Milvus, Chroma), GPUs, paid SaaS for inference or storage, async ORMs, Django.
*Why:* These are the wrong tools for the catalog sizes Findegil targets (≤1M documents). Adding them introduces operational complexity that this open-source project explicitly promises to avoid.

---

## 2. Latency Budgets (p99, single CPU core)

Findegil is fast by design. CI runs a benchmark against a frozen reference index. If any p99 exceeds its budget, the PR is blocked.

| Component | Budget |
|---|---|
| Normalization (unicode + lowercase + tokenize) | 0.5 ms |
| BM25 retrieval (Tantivy, top-100) | 3 ms |
| Embedding encode (e5-small INT8, 384d) | 5 ms |
| Cosine similarity (query vs full catalog matrix) | 1 ms |
| RRF merge + bitset filter | 1 ms |
| Feature computation (≤14 features) | 1 ms |
| CatBoost predict (~60 candidates) | 1 ms |
| JSON serialization | 0.5 ms |
| **Total /search p99 (cold)** | **15 ms** |
| **Total /search p99 (cached embedding)** | **2 ms** |
| Autocomplete (Trie lookup) | 0.5 ms |

*Note: Budgets are calibrated to the GitHub Actions `ubuntu-latest` runner.*

---

## 3. Architecture Invariants

These rules protect the system's runtime design.

- **A1 — Embedded lexical engine.** Tantivy runs in-process. No external search clusters.
- **A2 — In-RAM vector math.** Embeddings are a NumPy matrix in RAM. A NumPy matmul resolves similarity in <1 ms. No vector databases.
- **A3 — One master index + per-tenant bitsets.** Multi-tenancy is implemented via `numpy.packbits` AND-filtered against retrieval results. Adding a tenant is a tiny file write. Never create per-tenant indices.
- **A4 — CatBoost YetiRank is the standard.** It is the production LTR ranker. Do not export it to ONNX—the native `.cbm` runtime is faster and supports YetiRank's listwise scoring directly.
- **A5 — e5-small embeddings.** Larger models (e5-base) double encode latency for no measurable retrieval gain on short e-commerce queries.
- **A6 — Exact Typo Correction.** Typo correction happens via `fuzzy_query` against the live index, not via generic dictionaries like SymSpell (which produce spurious corrections for brand names).
- **A7 — No manual rule engines.** No pin/boost/demote layer modifying ranker output (except for legal/compliance hide-lists). Hand-tuned rules drift; anything relevant belongs in the feature spec so the ML model can weight it properly.

---

## 4. Machine Learning Invariants

When working on the training, evaluation, or LTR pipelines:

- **ML1 — IPW on every retrain.** Inverse Propensity Weighting on click position is mandatory. Without it, click position bias causes the model to reinforce its own past output, destroying diversity.
- **ML2 — Walk-forward CV.** Click data is temporal. Always train on past weeks, test on the future week. Random k-fold mixes future with past and inflates metrics.
- **ML3 — Frozen golden set.** A static set of queries with manual relevance judgments (like WANDS). **Add queries, never edit or delete existing judgments.** It is the only metric source immune to feedback-loop drift.
- **ML4 — `feature_spec.json` is the contract.** Features are defined once. CI enforces parity: training and serving must compute identical values for the same feature on the same input.
- **ML5 — Strict Reproducibility.** Every training run pins random seeds, git SHAs, and artifact hashes to a `manifest.json`. A retrain that cannot be replayed bit-for-bit is a bug.

---

## 5. Development & CI/CD Rules

Every PR must pass the following CI pipeline:
- **C1** — Latency benchmark (p99 budgets enforced).
- **C2** — Response schema contract test (no breaking API changes without versioning).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2701/findegil](https://github.com/2701/findegil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
