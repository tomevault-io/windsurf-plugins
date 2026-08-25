---
trigger: always_on
description: This repository implements a Nano-style information retrieval benchmark runner
---

# Repository Guidelines

## Project Overview

This repository implements a Nano-style information retrieval benchmark runner
for SentenceTransformers-compatible models and BM25 baselines.

Library code lives under `hakari_bench/`. Built-in dataset definitions live
under `config/datasets/`, and dataset collection definitions live under
`config/dataset_collections/`.

## Environment

- Use Python 3.12.
- Use `uv` for dependency management and command execution.
- Add runtime dependencies with `uv add`.
- Add development-only dependencies with `uv add --dev`.
- Keep `uv.lock` updated when dependencies change.
- Keep `transformers>=4` and `sentence-transformers>=5` compatibility.

## Development Workflow

- Follow TDD for behavioral changes: add or update focused tests before changing
  implementation when practical.
- Keep changes scoped to the existing module boundaries:
  - `cli.py` for command-line parsing and orchestration.
  - `datasets.py` for dataset specs, collections, split/task resolution.
  - `evaluation.py` for dataset loading and dense/sparse/reranker scoring.
  - `bm25.py` for BM25 baseline evaluation, candidate generation, and tokenizer
    dispatch.
  - `metrics.py` for IR metric calculation.
  - `models.py` for model loading and runtime/model metadata.
  - `results.py` for output paths, cache behavior, and JSON payloads.
- Prefer YAML dataset configuration over hard-coded dataset lists.
- Do not commit generated benchmark outputs, caches, or local scratch artifacts.
  `output/` and `tmp/` are intentionally ignored.

## Documentation

- Write project documentation in English unless the user explicitly requests
  another language.
- Keep reusable project workflows under `docs/`, not only under `skills/`.
  Skill files may point to the canonical docs, but they should not be the only
  place where benchmark or metadata procedures are described.
- Start with [`docs/index.md`](docs/index.md) when you need the human- and
  agent-facing map of canonical docs, workflow routes, and artifact boundaries.
- Use these docs as the main entry points when searching for repository
  procedures:

### General Orientation

- [`docs/quickstart.md`](docs/quickstart.md): shortest path from installation
  to first evaluation, DuckDB build, local viewer, and submission pointers.
- [`docs/benchmark_scope.md`](docs/benchmark_scope.md): compact overview of
  Nano-set task layout, coverage, dataset locations, and intended benchmark use.
- [`docs/index.md`](docs/index.md): grouped documentation map, source-of-truth
  table, workflow routes, and artifact commit policy.

### Evaluation And Model Workflows

- [`docs/evaluation_policy.md`](docs/evaluation_policy.md): canonical policy
  for prompts, dtype, attention, variants, BM25/reranker settings, runtime
  hygiene, cache policy, and coverage audits.
- [`docs/evaluation_runbook.md`](docs/evaluation_runbook.md): runnable commands
  for model evaluation, DuckDB rebuild/append, remote result sync, and local
  viewer startup.
- [`docs/new_model_results_workflow.md`](docs/new_model_results_workflow.md):
  end-to-end checklist for model research, model cards, validation, full
  evaluation, result PR body generation, and result submission.
- [`docs/contributing_results.md`](docs/contributing_results.md): result
  repository layout, `.json.xz` submission expectations, Hugging Face Dataset
  PR workflow, and reviewer checklist.
- [`docs/model_cards.md`](docs/model_cards.md): schema and workflow for static
  model metadata under `config/model_cards/`, including which settings belong in
  model cards.
- [`docs/model_specific_benchmarking_notes.md`](docs/model_specific_benchmarking_notes.md):
  model-family-specific prompts, runtime choices, compatibility notes, and
  reproducibility caveats.
- [`docs/custom_model_backends.md`](docs/custom_model_backends.md): custom
  loader contract for non-standard local or hosted dense, sparse, reranker, and
  late-interaction models.
- [`docs/late_interaction_evaluation.md`](docs/late_interaction_evaluation.md):
  PyLate/ColBERT late-interaction workflow, reviewed cards, prefixes, token
  lengths, and validation expectations.
- [`docs/openai_embedding_evaluation.md`](docs/openai_embedding_evaluation.md):
  OpenAI embedding setup, API dimensions, token limits, concurrency, and
  batch-evaluation pointers.
- [`docs/batch_inference.md`](docs/batch_inference.md): provider batch
  registration, output fetching, and materialization of normal HAKARI result
  JSON from offline embeddings.
- [`docs/sentence_transformers_evaluation_integration.md`](docs/sentence_transformers_evaluation_integration.md):
  SentenceTransformers training-time evaluators, Nano target selection, smoke
  runs, and integration tests.

### Dataset, Metadata, And Public Task Documentation

- [`docs/create_nano_datasets.md`](docs/create_nano_datasets.md): Nano dataset
  creation, MTEB-derived Nano family recreation, BM25/dense top-500 candidates,
  and reranking-hybrid top-100 safeguards.
- [`docs/NanoREADME.template.md`](docs/NanoREADME.template.md): generated
  Hugging Face dataset card template for Nano dataset layout, construction,
  candidate subsets, and quality tables.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hakari-bench/hakari-bench](https://github.com/hakari-bench/hakari-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
