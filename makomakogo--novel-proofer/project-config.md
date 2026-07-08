---
trigger: always_on
description: Novel Proofer is a local Chinese novel formatting/proofreading tool. It has a FastAPI backend in `novel_proofer/` and a static HTML/Tailwind frontend in `templates/`.
---

# Agent Guide

## Project Overview

Novel Proofer is a local Chinese novel formatting/proofreading tool. It has a FastAPI backend in `novel_proofer/` and a static HTML/Tailwind frontend in `templates/`.

It applies deterministic local rules for punctuation, indentation, whitespace, and CJK punctuation normalization, and uses an OpenAI-compatible LLM for semantic formatting such as paragraph splitting, dialogue separation, chapter-title handling, and first-chunk ad/watermark cleanup.

## Setup And Run

Use `uv`; do not install Python packages globally.

```bash
# Sync dev dependencies
uv sync --frozen --no-install-project --group dev

# Run the server
uv run --frozen --no-sync -m novel_proofer.server

# One-command startup
start.bat
bash start.sh
```

The server runs at `http://127.0.0.1:18080` by default unless the startup script chooses another available port.

## Validation

Run these before handing off code changes:

```bash
uv run --frozen --no-sync ruff format --check
uv run --frozen --no-sync ruff check
uv run --frozen --no-sync python -m mypy novel_proofer
uv run --frozen --no-sync python -m pytest -q
uv run --frozen --no-sync python tools/export-openapi.py --check
uv run --frozen --no-sync python tools/check-large-files.py
```

`pytest` addopts in `pyproject.toml` enforce coverage, branch coverage, marker strictness, and slow-test duration reporting.

If API routes or response models change, regenerate/check the schema with:

```bash
uv run --frozen --no-sync python tools/export-openapi.py
```

If Tailwind classes change in `templates/index.html` or `templates/static/js/**`, run:

```bash
npm ci
npm run build:css
```

## Architecture

```
novel_proofer/
├── server.py       # Entry point: uvicorn CLI wrapper
├── api.py          # FastAPI app, REST endpoints, request validation
├── jobs.py         # JobStore: thread-safe job/chunk state management
├── executions.py   # In-process execution registry and stop requests
├── workflow.py     # Workflow guards and invariants
├── runner.py       # Orchestrator: chunking -> local rules -> LLM -> merge
├── background.py   # Bounded job-level thread pool
├── models.py       # Pydantic request/response models
├── converters.py   # API conversion helpers
├── formatting/
│   ├── config.py   # FormatConfig dataclass
│   ├── rules.py    # Deterministic text transformations
│   ├── chunking.py # Split text by line boundaries
│   ├── merge.py    # Merge processed chunks
│   └── fixer.py    # Legacy/utility formatters
└── llm/
    ├── config.py   # LLMConfig and system prompts
    ├── client.py   # OpenAI-compatible streaming client with retry logic
    └── think_filter.py
```

## Data Flow

1. Upload (`POST /api/v1/jobs`): cache decoded UTF-8 input and create job status.
2. Validate phase: split text by line boundaries and apply deterministic local rules.
3. Process phase: call the LLM concurrently per chunk, with retry and output validation.
4. Post-process: re-apply local rules to LLM output for consistency.
5. Merge phase: combine chunk outputs and write final output under `output/`.

Runtime and generated data live under `output/`:

- `output/.inputs/{job_id}.txt`: input cache for reruns/recovery.
- `output/.jobs/{job_id}/`: per-job debug/intermediate artifacts.
- `output/.state/jobs/{job_id}.json`: job state snapshots.
- `output/{job_id}_...txt`: final merged output.

Keep generated/runtime data out of commits: `.env*` except examples, `output/`, caches, and `tests/.artifacts/` are ignored.

## Key Concepts

- Job states: `queued`, `running`, `paused`, `done`, `error`, `cancelled`.
- Execution state is volatile and process-local; it lives in `executions.py`, not in persisted job records.
- Job phases: `validate`, `process`, `merge`, `done`.
- Chunk states: `pending`, `processing`, `retrying`, `done`, `error`.
- `retrying` is an automatic retry/backoff state within a single chunk processing attempt, not a manual retry queue.
- `reset` is the hard delete/cleanup action for a job. It should not be confused with UI detach/new task.

## Workflow Architecture

The current workflow model is documented in `docs/WORKFLOW.md`, `docs/STATE_MACHINE.md`, and `docs/ARCHITECTURE.md`:

- `JobRecord`: durable task and artifact truth.
- `JobExecution`: volatile in-process worker attempt.
- `UiAttachment`: browser-side association with a job id.

Do not add compatibility shims or silent fallback behavior to workflow code.

## Local Rules Reference

Rules in `formatting/rules.py` run in this order:

1. Normalize newlines.
2. Trim trailing spaces.
3. Collapse multiple blank lines.
4. Normalize ellipsis.
5. Normalize em dash.
6. Convert ASCII punctuation to fullwidth in CJK context.
7. Remove spaces between CJK and punctuation.
8. Convert straight quotes to curly in CJK lines when enabled.
9. Apply paragraph indent, skipping chapter titles.

## API Endpoints

- `POST /api/v1/jobs` - Create job.
- `POST /api/v1/jobs/{id}/rerun-all` - Re-create job from the same input with new options.
- `GET /api/v1/jobs` - List jobs.
- `GET /api/v1/jobs/{id}` - Get job status and chunks.
- `GET /api/v1/jobs/{id}/input-stats` - Get cached input stats.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makoMakoGo/novel-proofer](https://github.com/makoMakoGo/novel-proofer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
