---
trigger: always_on
description: Python implementation patterns inferred from OASIS refactor commits
---


# OASIS Python Architecture

- **No duplicated logic (non-negotiable)**: **Never** copy-paste or parallel-implement the same behavior in two places. If something exists twice (even slightly divergent), **stop** and consolidate: one canonical implementation, callers import or delegate to it. This is mandatory DRY; KISS applies to the **solution**, not to skipping extraction. SOLID implies a single place per responsibility—duplication always violates that.
- **Centralize by default**: Shared constants, validation rules, formatters, parsers, JSON/report field naming, URL or path rules, and error messages belong in one module each (`oasis/helpers/`, `oasis/schemas/`, or another existing shared module)—not redefined ad hoc in multiple files.
- Keep responsibilities split by module: orchestration in `oasis/oasis.py`, analysis in `oasis/analyze.py`, structured schemas in `oasis/schemas/`, canonical JSON + Jinja report rendering in `oasis/report.py`, dashboard indexing in `oasis/web.py`, model lifecycle in `oasis/ollama_manager.py`.
- **LangGraph (`oasis/agent/`)**: The compiled DAG (`graph.py`), state (`state.py`), stable node/route identifiers (`graph_labels.py`), and `invoke_oasis_langgraph` (`invoke.py`) own orchestration structure. Node bodies dispatch to `SecurityAnalyzer` methods in `analyze.py` via `tools.py` / `nodes.py` — keep business logic in `analyze.py` and keep the graph layer a thin wiring + state pass-through. Use explicit imports from submodules (e.g. `oasis.agent.invoke`) so importing `oasis.agent` does not eagerly load LangGraph unless needed.
- **Analysis type**: `AnalysisType.GRAPH` is the only scan/deep orchestration mode; chunk caches use LangGraph paths only (`oasis/cache.py`).
- **LangGraph-related helpers**: `oasis/helpers/langgraph_cli/` bundles CLI banners/emits, debug separators, LLM debug logging, and LangGraph vuln-type count helpers. `oasis/helpers/progress/` includes graph pipeline rows (`graph_pipeline_phases`, `graph_progress_extras`, …), tqdm/coercion, executive-summary phase extras, and scan progress JSON/markdown helpers. `oasis/helpers/context/expand.py` holds `expand_line_window` / `expand_suspicious_chunk_records` (`CONTEXT_EXPAND_*` in `oasis/config.py`). `oasis/helpers/poc/` covers PoC digest JSON, hints markdown, and PoC stage logging options. `oasis/helpers/ollama_timing/` holds Ollama payload size / timeout helpers for logging.
- **Embedding model normalization**: Parse and normalize CLI embedding model values through `oasis/helpers/embedding/` (`normalize_embed_models`, `resolve_embed_models`, `primary_embed_model`, `resolve_valid_embedding_input_files`) and keep one canonical source for model-list parsing and primary-model fallback. Invalid input raises `EmbedModelValueError` (subclass of `ValueError`); `oasis/oasis.py` maps that to `argparse.ArgumentTypeError` only in `argparse` `type=` callables.
- **Helpers (`oasis/helpers/`)**: All reusable utilities that behave like helpers **must** live under `oasis/helpers/`, not inlined in feature modules (`analyze.py`, `report.py`, `web.py`, etc.). If you add or refactor something that is formatting, parsing, small pure transforms, progress/status builders, shared guards, or other cross-cutting non–entry-point utilities, **move it** into `oasis/helpers/` as part of the change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psyray/oasis](https://github.com/psyray/oasis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
