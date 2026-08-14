---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`Ultimate Macroeconomics Dashboard` is a Docker stack: a **multi-page Streamlit dashboard** (`app/`) backed by Postgres + Qdrant, with FastAPI micro-services for the AI analyst, forecasting, clustering, on-demand data ingestion, a sandboxed Python executor, and document conversion, plus an NVIDIA Triton Inference Server that hosts all forecasting/clustering model inference (GPU where possible, CPU python backend otherwise). The Streamlit app talks **directly** to the backend services (the read-only Postgres role, Qdrant, and the agent / forecaster / clustering HTTP APIs) — there is no backend-for-frontend. Read `README.md` for the full description; the sections below cover only what isn't obvious from the code.

Planning docs: `TODO.md` is the flat, actionable backlog of remaining work.

## Running the stack

Linting/formatting is done with `ruff` and type-checking with `ty` (both Astral); each service ships a `[dependency-groups] dev` block with both plus `pytest` and `pytest-cov`. Ruff and ty are configured **once at the repo root** in `ruff.toml` and `ty.toml` (single source of truth) — the per-service `pyproject.toml` files intentionally carry no `[tool.ruff]`/`[tool.ty]` tables, so running `ruff`/`ty` from any service directory walks up and resolves the root config (each service's own `.venv` is still used for import resolution). The root `ruff.toml` turns on import sorting (isort `I` with `combine-as-imports` + `force-sort-within-sections`) plus formatter niceties (`lf` line endings, docstring code formatting). Tests live under `<service>/tests/` and run via `uv run pytest` — `[tool.pytest.ini_options].addopts` enables coverage by default (`--cov --cov-report=term-missing`), with per-service `[tool.coverage.run].source` pointing at that service's package(s). Everything runs inside containers via Docker Compose. Every service is Python 3.12 and uses [uv](https://docs.astral.sh/uv/) for dependency management — each service has its own `pyproject.toml` + `uv.lock`, and the Dockerfile runs `uv sync --frozen` into `/opt/venv`.

```bash
# Full stack (build + run, foreground)
docker compose up --build

# Single service rebuild
docker compose build agent
docker compose up -d agent

# Logs
docker compose logs -f app
docker compose logs -f agent
```

For local iteration without rebuilding the image, work in any service directory:

```bash
cd agent          # or forecaster, downloader_extra, docling, etc.
uv sync           # creates .venv from pyproject.toml + uv.lock
uv run uvicorn main:app --reload   # FastAPI services
uv add <package>      # add a dependency (updates pyproject.toml + uv.lock)
uv lock --upgrade     # refresh the lockfile
```

The **Streamlit dashboard** (`app/`) is a uv-managed Python service like the rest. Work in `app/`:

```bash
cd app
uv sync
uv run streamlit run app.py   # dev server on :8501 (talks directly to the services)
uv run pytest                 # unit tests
# end-to-end (Playwright, separate pnpm project): cd app/e2e && pnpm install && pnpm exec playwright test
```

First boot requires `_container_data/.env` (copy from `_container_data/.env.example`) and a populated LLM section in `_container_data/config.yaml`. The `postgres:18` image creates the superuser (`POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB`) natively on first volume init; `downloader_general` then upserts the read-only LLM role via `src/utils/db_bootstrap.py` (also grants `SELECT` on `public` plus default privileges, so future tables are readable automatically) and runs the ingestion (~1–2h) for World Bank + Yahoo Finance + Webz.io news. The dashboard is at `http://localhost:8501`.

The bootstrap step runs on **every** `downloader_general` container start (cheap, idempotent), so rotating `POSTGRES_LLM_PASSWORD` or adding new tables in a future release takes effect on the next `docker compose up -d downloader_general` without wiping volumes. Only the **initial** bulk download is one-shot, gated by `_container_data/downloader_general/.download_completed`. After that marker is written the container **does not exit** — it stays up running a per-source incremental update scheduler (see the `scheduler:` block in `config.yaml` and the "Data ingestion" section). Because the container is now long-running, its Compose `depends_on` gate flipped from `service_completed_successfully` to `service_healthy` (the healthcheck just tests for the marker file), and `restart:` is `unless-stopped`.

A working NVIDIA GPU + the NVIDIA Container Toolkit are **required**: all model inference runs inside the `triton` service, which reserves the GPU via its `deploy:` block in `docker-compose.yaml`. Chronos + XGBoost run on CUDA, clustering/dim-reduction on RAPIDS cuML, and the CPU-only models (ARIMA family, Prophet, moving-average) run on Triton's python CPU backend. There is no CPU-only fallback build — the `forecaster`/`clustering` containers are now GPU-free adapters, and the GPU reservation lives on `triton` alone.

## Architecture

### Service map and ports


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleksey-karasev/Ultimate-Macroeconomics-Dashboard](https://github.com/aleksey-karasev/Ultimate-Macroeconomics-Dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
