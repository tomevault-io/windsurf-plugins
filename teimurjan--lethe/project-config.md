---
trigger: always_on
description: Self-improving memory store for LLM agents. BM25 + dense vector hybrid
---

# lethe — agent guide

Self-improving memory store for LLM agents. BM25 + dense vector hybrid
retrieval with cross-encoder reranking and clustered retrieval-induced
forgetting (RIF). Indexes agent transcripts (Claude Code / Codex) directly
— no capture hooks, no summarization. Ships as a single Rust binary (`lethe`)
plus PyPI / npm bindings.

## stack

- **Rust workspace** (`crates/`) — production. Toolchain pinned via
  `rust-toolchain.toml`.
  - `lethe-core` — library: tokenize, bm25, faiss_flat, rrf, dedup, rif,
    kmeans, encoders (ONNX via `ort`), DuckDB persistence, npz reader,
    memory_store, union_store, markdown_store, transcript_store.
  - `lethe-cli` — `lethe` binary (clap; embeds the TUI).
  - `lethe-tui` — ratatui library called from the CLI on no-arg invocation.
  - `lethe-py` — PyO3 binding → PyPI `lethe-memory`.
  - `lethe-node` — napi-rs binding → npm `lethe`.
  - `lethe-benchmark` — internal parity bench helper (`publish = false`).
- **DuckDB** for entry metadata + embedding BLOBs (single source of truth).
- **ONNX Runtime** (via `ort`) for the bi-encoder + cross-encoder.
  Default models: `Xenova/all-MiniLM-L6-v2`, `Xenova/ms-marco-MiniLM-L-6-v2`.
- **Python `research_playground/lethe_reference/`** kept for the research-journey trail: produces the
  benchmark numbers cited below. Installed as `lethe-memory-legacy` in
  dev venvs; not published.
- **Datasets** (`tmp_data/`, gitignored): LongMemEval (S), NFCorpus.

## commands

```bash
# Rust workspace
cargo build --workspace --release        # → target/release/lethe
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all -- --check

# Pre-commit / pre-push hooks (replaces the deleted ci.yml)
cargo install --git https://github.com/j178/prek
prek install                             # writes .git/hooks/pre-commit + pre-push
prek run --all-files                     # one-off run

# Python reference impl (research trail; library only)
uv pip install -e research_playground/lethe_reference/
cd research_playground/lethe_reference && uv run pytest tests/ -v     # 148 + 8 PyO3 parity = 156
uv run python research_playground/baseline/run.py

# Python ↔ Rust parity bench
uv run python research_playground/rust_migration/prepare.py
uv run python research_playground/rust_migration/longmemeval.py --compare
uv run python research_playground/rust_migration/components.py --compare
uv run python research_playground/rust_migration/latency.py --compare

# CLI surface (the `lethe` binary)
lethe                                    # no args → TUI (in a terminal)
lethe index                              # index this project's transcripts
lethe index --all                        # reindex every registered project
lethe search "query" --top-k 5           # reindexes changed transcripts first
lethe search "query" --all --top-k 5     # all registered projects (per-project read-only)
lethe projects list|add|remove|prune
lethe expand <chunk-id> [<chunk-id> ...]
lethe status
```

`tree -L 2 -I 'target|.venv|node_modules|tmp_*|results|.git|.lethe'` if
you want the current directory layout.

## key architecture decisions

- BM25 + FAISS-equivalent hybrid retrieval (BM25 is the strongest single
  signal on conversation data).
- Cross-encoder rerank on the merged candidate pool; adaptive depth
  (shallow `k=30`, deep `k=100` only when shallow confidence is low —
  see `research_playground/deep_pass/results/BENCHMARKS_DEEP_PASS.md`).
- Cosine 0.95 dedup on add (removes ~5% of LongMemEval, +6.5% NDCG).
- Tier lifecycle: naive → gc → memory (with decay and apoptosis).
- RIF: retrieval-induced forgetting suppresses chronic false positives
  at the candidate-selection stage (workload-specific to long-term
  conversational memory).
- DuckDB + npz interop for persistence; no external services.
- Embeddings are *the* source of truth in DuckDB. `MemoryStore::open()`
  rebuilds the in-memory `FlatIp` from BLOBs every cold start
  (~100 ms on a 200k corpus); no separate FAISS index file is written.

## benchmark headline

LongMemEval S (200k turns, 500 questions, 200-query eval sample):

| System | NDCG@10 |
|---|---|
| Vector only | 0.1376 |
| BM25 only | 0.3171 |
| Hybrid RRF | 0.2408 |
| **Hybrid + cross-encoder rerank** | **0.3817** |

Full methodology: `BENCHMARKS.md`. 18-checkpoint research journey:
`RESEARCH_JOURNEY.md`. arXiv preprint source: `paper.tex`.

## docs / READMEs

The root `README.md` is the project landing page (linked from each
per-crate README). Each publishable crate also has its own README
that gets surfaced on the registry it ships to:

| File | Surfaces on |
|---|---|
| `crates/lethe-core/README.md` | crates.io `lethe-core` |
| `crates/lethe-cli/README.md`  | crates.io `lethe-cli` |
| `crates/lethe-tui/README.md`  | crates.io `lethe-tui` |
| `crates/lethe-py/README.md`   | PyPI `lethe-memory` |
| `crates/lethe-node/README.md` | npm `lethe` |
| Root `README.md`              | GitHub repo landing page |

When you edit the root `README.md`, **always check whether the change
should propagate to one or more per-crate READMEs.** Examples that
need to fan out:

- Install instructions changed → update `lethe-cli/README.md`,
  `lethe-py/README.md`, `lethe-node/README.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teimurjan/lethe](https://github.com/teimurjan/lethe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
