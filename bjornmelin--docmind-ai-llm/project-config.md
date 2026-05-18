---
trigger: always_on
description: Repo guardrails for contributors/automation. Keep aligned with code, `pyproject.toml`,
---

# DocMind AI - Agent Instructions

## Purpose

Repo guardrails for contributors/automation. Keep aligned with code, `pyproject.toml`,
and docs under `docs/specs/` + `docs/developers/adrs/`.

## Layout

- `app.py`: Streamlit entrypoint
- `src/app.py`: Streamlit app module (imported by `app.py`)
- `src/pages/`: UI pages (chat/documents/analytics/settings)
- `src/config/`: settings + integration wiring
- `src/processing/`: ingestion, OCR, PDF page exports
- `src/retrieval/`: router, hybrid retrieval, reranking, GraphRAG helpers
- `src/agents/`: LangGraph coordinator (graph-native supervisor via `StateGraph`)
- `src/persistence/`: snapshots, hashing, locking, chat DB
- `src/telemetry/` + `src/utils/telemetry.py`: OTEL + JSONL events
- `templates/`: prompt templates/presets
- `docs/specs/` + `docs/developers/adrs/`: specs/ADRs (source-of-truth docs)
- `scripts/` + `tools/`: dev utilities (tests, perf, model pull)

## Quick Commands (uv)

- Setup: `uv sync && cp .env.example .env`
- Run: `uv run streamlit run app.py` (or `./scripts/run_app.sh`)
- Env: prefer `uv run ...` (uses the project env, typically `.venv`).
- Verify (batch): after a batch of edits, run lint/type on touched paths + focused tests.
  - Lint (all): `uv run ruff format . && uv run ruff check . --fix`
  - Type (paths): `uv run pyright --threads 4 <paths>`
  - Tools-only (when `tools/` changed): `uv run pyright --threads 4 tools`
  - Tests (focused): `uv run pytest <tests/...> -vv` (or `-k <expr>` for a narrow slice)
- Verify (final): before finishing the task/prompt, run full lint/type then full tests: `uv run ruff format . && uv run ruff check . --fix && uv run pyright --threads 4 && uv run python scripts/run_tests.py`
- Tests (fast): `uv run python scripts/run_tests.py --fast`
- Coverage: `uv run python scripts/run_tests.py --coverage`
- Coverage report: `uv run python scripts/check_coverage.py --collect --report --html`
- Quality gates (CI): `uv run python scripts/run_quality_gates.py --ci --report`
- Perf check: `uv run python scripts/performance_monitor.py --run-tests --check-regressions --report`
- GPU check: `uv run python scripts/test_gpu.py --quick`
- Prefetch models: `uv run python tools/models/pull.py --all --cache_dir ./models_cache`
- spaCy model (opt): `uv run python -m spacy download en_core_web_sm`
- Review triage: `uv run python scripts/analyze_github_reviews.py --json-file <path>` (or set `DOCMIND_REVIEW_JSON`)

## Non-negotiables (CI + Security)

- No `TODO|FIXME|XXX` under `src tests docs scripts tools`.
- CI expects `ruff format --check` and clean `ruff check` (CI runs `ruff check --fix --exit-non-zero-on-fix`).
- Offline-first default: no implicit egress. Base URL validation is strict by default and includes DNS resolution of allowlisted non-loopback hosts as SSRF/DNS-rebinding hardening.
- Streamlit: no `unsafe_allow_html=True` for untrusted content.
- Logging/telemetry: metadata-only; never log secrets or raw prompt/doc/model output (use `src/utils/log_safety.py`).

## Compaction + continuity worklogs

This repo uses a lightweight, compaction-resilient log so we can resume work without re-researching or re-deciding.

Worklogs are **local-only** by default (gitignored) and should not be committed to the repo.

After any material research, decisions, or implementation:

1. Update `docs/developers/worklogs/CONTEXT.md` with:
   - current status + next steps
   - research notes (primary links)
   - key decisions + rationale
   - important quirks/constraints (no secrets)
2. If you call any `mcp__zen__*` tool that returns a `continuation_id`, record it in:
   - `docs/developers/worklogs/continuations.json`
3. For normative changes (architecture/policy): add or update an ADR under `docs/developers/adrs/`.
4. For behavior changes: update the owning spec under `docs/specs/` and keep `docs/specs/traceability.md` aligned.

Timing rule: write ADRs/spec updates **immediately when finalized** (do not batch them at the end of a long session) to avoid losing decision context during auto-compaction.

When resuming after compaction:

1. Read `docs/developers/worklogs/CONTEXT.md` first.
2. Read `docs/developers/worklogs/continuations.json` and reuse any stored `continuation_id` values when continuing `mcp__zen__*` threads.

## Optional extras

- `uv sync --frozen --extra gpu` (fastembed-gpu, CuPy CUDA wheels)
- `uv sync --frozen --extra graph` (GraphRAG adapters)
- `uv sync --frozen --extra multimodal` (ColPali reranker)
- `uv sync --frozen --extra observability` (OTLP exporters + portalocker)
- `uv sync --frozen --extra eval` (beir)

## Dependency constraints (don’t drift)

Source of truth for exact pins: `pyproject.toml` + `uv.lock`.

- Python: `>=3.12,<3.14` (primary dev/runtime: Python 3.12.13)
- Keep these coupled:
  - Torch 2.8.x ↔ Transformers `>=5.0,<6.0` (vLLM is external-only via OpenAI-compatible HTTP)
  - DuckDB `<1.4.0` (LlamaIndex integrations cap it)
  - LlamaIndex packages stay `<0.15.0`
  - Streamlit `<2.0.0`
  - `[tool.uv]` enforces `rapidfuzz>=3.14.1,<4.0.0`

## Configuration

- Source of truth: `src/config/settings.py` (Pydantic Settings v2).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BjornMelin/docmind-ai-llm](https://github.com/BjornMelin/docmind-ai-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
