---
trigger: always_on
description: A Python library for benchmarking Weaviate's Query Agent. It supports two evaluation modes:
---

# CLAUDE.md (AI-generated overview)

## Project Overview

A Python library for benchmarking Weaviate's Query Agent. It supports two evaluation modes:
- **Search mode**: Ranked retrieval evaluation using IR metrics (Recall@K, nDCG@K, Coverage, alpha-nDCG)
- **Ask mode**: Question answering evaluation using LLM-as-judge (DSPy-based ensemble voting for semantic alignment)

## Commands to Run

```bash
uv sync                                        # Install dependencies
uv run python3 scripts/populate-db.py           # Populate Weaviate with benchmark data
uv run python3 scripts/run-search-benchmark.py  # Run search benchmark
uv run python3 scripts/run-ask-benchmark.py     # Run ask benchmark
uv run python3 scripts/run-compare-embeddings.py # Compare embedding models

# Start the toy benchmark server
uv run uvicorn query_agent_benchmarking.cmd.server:app --reload

# Run tests (unit/integration only — fast, no network)
uv run pytest -v

# Run functional tests (downloads real data)
uv run pytest tests/functional/ -v

# Run Weaviate integration tests (requires WEAVIATE_URL + WEAVIATE_API_KEY)
uv run pytest tests/functional/test_database_weaviate.py -v
```

## Environment Variables

Configured via `.env` at the project root (loaded automatically via `python-dotenv`). See `.env.example` for the full list.

---

## Hexagonal Architecture

This package is organized using the **hexagonal architecture** pattern, also known as "ports and adapters." The central idea is that the domain logic — the rules, models, and workflows that define what the system *does* — should never depend on infrastructure details like which database you're talking to, which HTTP client you're using, or how files get written to disk. Instead, the domain declares abstract interfaces (called **ports**) that describe what capabilities it *needs*, and concrete implementations (called **adapters**) plug into those ports from the outside. This inverts the traditional dependency direction: infrastructure depends on the domain, never the other way around. The payoff is that the core logic becomes testable in isolation (swap in a mock adapter), extensible without modification (add a new adapter for a new database), and readable on its own terms (the domain code says *what* without drowning in *how*).

### How It Works in This Package

The package is split into layers inside `query_agent_benchmarking/internal/`. The heart of the architecture lives in `core/`, which is itself divided into three packages that correspond to the three rings of the hexagonal model:

```
internal/
├── core/                  # The hexagonal core
│   ├── domain/            # Innermost ring — pure logic, zero adapter imports
│   │   ├── models.py              # Pydantic data contracts (InMemoryQuery, QueryResult, etc.)
│   │   ├── metrics_config.py      # Dataset-to-metrics mapping (data only, no function refs)
│   │   ├── query_execution.py     # Query runner logic (iteration, batching, concurrency)
│   │   ├── analysis.py            # Cross-trial metric aggregation (pure math)
│   │   └── benchmark_orchestrator.py  # DI-wired orchestrators
│   │
│   ├── ports/             # Abstract interfaces the domain needs
│   │   ├── search_agent.py        # SearchAgent protocol
│   │   ├── ask_agent.py           # AskAgent protocol + AskResponse
│   │   ├── dataset_repository.py  # SearchDatasetRepository, AskDatasetRepository
│   │   ├── metrics_calculator.py  # SearchMetricsCalculator, AskMetricsCalculator
│   │   ├── result_repository.py   # ResultRepository
│   │   ├── database_manager.py    # DatabaseManager
│   │   └── llm_judge.py           # LLMJudge
│   │
│   └── services/          # Application layer — wires adapters to domain
│       ├── search_benchmark.py    # run_search_eval, run_search_evals
│       ├── ask_benchmark.py       # run_ask_eval
│       ├── populate_db.py         # populate_db
│       └── compare_embeddings.py  # compare_embeddings
│
├── adapters/       # Concrete implementations that plug into ports
│   ├── agents/         # SearchAgent & AskAgent implementations + factory
│   │   ├── factory.py            # create_search_agent(), create_ask_agent()
│   │   ├── weaviate_query_agent.py  # Weaviate QueryAgent (search + ask)
│   │   ├── weaviate_search.py    # Hybrid, vector, BM25 search
│   │   ├── external_service.py   # HTTP-based BYOS evaluation
│   │   └── engram_dspy_agent.py  # Engram + DSPy for LongMemEval
│   ├── clients/        # Weaviate client factory, provider header resolution
│   ├── database/       # Collection creation, batch insert, dataset specs
│   ├── dataset/        # Data loaders (HuggingFace, ir_datasets, Weaviate, local)
│   ├── metrics/        # IR metrics, LLM judge, exact match, OfficeQA
│   └── results/        # JSON file persistence
│
├── config/         # YAML config loading, dataset/metric registries, system prompts
├── mocks/          # No-op implementations for testing
└── testutil/       # Query/result factory helpers for tests
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weaviate/query-agent-benchmarking](https://github.com/weaviate/query-agent-benchmarking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
