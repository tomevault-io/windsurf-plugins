---
trigger: always_on
description: High-throughput, batched OpenAI / Azure OpenAI Responses + Embeddings for pandas & Spark with strict ordering, deduplication, and structured outputs.
---

# Copilot Instructions – openaivec

High-throughput, batched OpenAI / Azure OpenAI Responses + Embeddings for pandas & Spark with strict ordering, deduplication, and structured outputs.

---

## Dev Commands

All tooling runs via **uv**. Python ≥ 3.10.

```bash
uv sync --all-extras --dev          # install deps + dev tools
uv run ruff check . --fix           # lint (auto-fix)
uv run ruff format .                # format
uv run pyright src/openaivec        # type check
uv run pytest -q                    # full test suite
uv run pytest tests/test_responses.py::test_reasoning_temperature_guard -q  # single test
uv run pytest -m "not slow and not requires_api"  # fast iteration without API keys
uv run mkdocs serve                 # local docs
uv build                            # validate distribution
```

Environment: set `OPENAI_API_KEY`, or for Azure set `AZURE_OPENAI_API_KEY` + `AZURE_OPENAI_BASE_URL` (must end with `/openai/v1/`) + `AZURE_OPENAI_API_VERSION` (`"preview"`). Tests auto-skip when credentials are absent.

---

## Architecture

### Public surface

| Layer | Entry point | Notes |
|-------|------------|-------|
| Package exports | `openaivec.__init__` | `BatchResponses`, `AsyncBatchResponses`, `BatchEmbeddings`, `AsyncBatchEmbeddings`, `PreparedTask`, `FewShotPromptBuilder`, `FewShotPrompt`, `SchemaInferer`, `SchemaInferenceInput`, `SchemaInferenceOutput` |
| pandas accessors | `Series.ai` / `Series.aio` | Sync + async; registered by importing `openaivec.pandas_ext` |
| Spark UDFs | `openaivec.spark_ext` | `responses_udf`, `task_udf`, `embeddings_udf`, `count_tokens_udf`, `split_to_chunks_udf`, `similarity_udf`, `parse_udf`, `infer_schema` |
| DuckDB UDFs | `openaivec.duckdb_ext` | `responses_udf`, `embeddings_udf`, `task_udf`, `similarity_search`, `pydantic_to_duckdb_ddl`, `DuckDBCacheBackend` |
| Task factories | `openaivec.task.nlp`, `.customer_support`, `.table` | Call as functions: `nlp.sentiment_analysis()`, not constants |
| Schema inference | `openaivec._schema` | `SchemaInferer` infers Pydantic models from sample data; used by `parse` helpers when `response_format=None` |

### Data flow

```
User input (list / Series / Spark column)
  → BatchCache (_cache/proxy.py)    # dedup, order-preserve, mini-batch
    → _responses.py / _embeddings.py      # OpenAI API with backoff
  → results reassembled in original order
```

`BatchCache` / `AsyncBatchCache` in `_cache/` are the core execution engine. They deduplicate inputs, chunk into batches, call a `map_func`, and restore original ordering. The `map_func` **must** return a list of identical length and order — a mismatch raises `ValueError` after releasing in-flight waiters (deadlock prevention).

`batch_size=None` enables `BatchSizeSuggester` (`_cache/optimize.py`) auto-tuning that targets ~30–60s per batch. Positive values force fixed chunks; `<= 0` processes everything in one call.

The cache layer is pluggable via `CacheBackend` (default: in-memory `OrderedDict`). `DuckDBCacheBackend` provides persistent cross-session caching. Reuse caches from `*_with_cache` helpers per operation and clear them (`clear`/`aclose`) when finished to avoid unbounded growth.

### Internal vs public boundary

Underscore-prefixed modules (`_responses.py`, `_cache/`, `_schema/`, `_di.py`, etc.) are internal — set `__all__ = []`. Public modules: `pandas_ext/`, `spark_ext.py`, `task/`, and `__init__.py`.

---

## Key Contracts

1. **Batch everything** — all remote calls go through the proxy; never per-item API loops.
2. **Same-length invariant** — `map_func` output must match input length and order exactly.
3. **Dedup + restore** — duplicate inputs are collapsed; outputs are expanded back to original positions.
4. **Preserve pandas index / Spark schema** — no hidden reindexing or sorting.
5. **Reasoning models** (o1/o3 families) — must set `temperature=None`.
6. **Exponential backoff** — `@backoff` / `@backoff_async` for `RateLimitError` / `InternalServerError`, max 12 retries.
7. **Structured outputs preferred** — use Pydantic `response_format` over free-form JSON/text.
8. **Progress bars** — only in notebooks and only when `show_progress=True`.

---

## Coding Conventions

- **Ruff** lint + format, `line-length=120`, target `py310`.
- **Absolute imports only** — enforced by Ruff TID252. Exception: `__init__.py` may use relative imports for re-exports.
- **Modern typing** — `list[T]`, `dict[K, V]`, `X | None` (not `Optional`), `collections.abc.Callable` (not `typing.Callable`).
- **`@dataclass`** for all classes — including wrappers, backends, and caches. Every field must have an explicit type annotation. Use **Pydantic** only at validation boundaries (API responses, task models).
- **Dependency injection** — never create collaborators inside `__init__` / `__post_init__`. Accept them as typed dataclass fields so they can be swapped or mocked. Provide a `@classmethod of(...)` factory for convenient construction with sensible defaults.
- **Narrow exceptions** — `ValueError`, `TypeError` on contract violations; no broad `except`.
- **Google-style docstrings** — Args with `(type)` annotations, Returns/Raises sections.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/openaivec](https://github.com/microsoft/openaivec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
