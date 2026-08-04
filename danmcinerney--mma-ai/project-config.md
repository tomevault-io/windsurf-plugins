---
trigger: always_on
description: This file is the root instruction surface for coding agents working in this
---

# MMA AI Agent Guide

This file is the root instruction surface for coding agents working in this
repository. Keep it high-signal and repo-specific. The detailed human-facing
reference lives in `README.md`; use this file for rules that should change how
an agent edits, tests, or reasons about the code.

## Project Snapshot

MMA AI is a public, Dockerized UFC analytics and prediction app. It combines the
historical UFCStats scraper from `UFCScraper` with the PostgreSQL feature store,
training, and prediction pipeline from `mma-ai-db`.

The release surface is:

- A FastAPI dashboard for data refresh, read-only analytics, and prediction.
- CLI training and evaluation for advanced users.
- Docker Postgres 18.1 with two databases: `mma-ai` and `odds`.

Do not add dashboard training controls or a training navigation surface.
Training remains a CLI workflow.

## Repository Map

- `libs.web.app:app`: web app entry point.
- `libs/web`: FastAPI routes, services, jobs, analytics, evaluation summaries,
  path safety, and static UI.
- `libs/web/static`: dashboard frontend. Charts load from `/vendor/plotly.min.js`
  and icons load from `libs/web/static/icons.js` through a local Lucide shim.
  Do not add public CDN dependencies for Plotly or icons.
- `libs/scraping/ufcstats.py`: in-repo UFCStats scraper adapter.
- `libs/feature_store`: PostgreSQL schema, calculators, feature table assembly,
  training-data creation, and inference feature builders.
- `libs/modeling`: training, walk-forward utilities, evaluation, calibration,
  profit reporting, and portable artifacts.
- `scripts`: CLI adapters and release/dev utilities.
- `docs/ANALYTICS_SCHEMA.md`: plain-English analytics schema reference for
  feature meanings, leakage status, odds units, and query patterns.
- `data/raw/ufcstats`: tracked seed `competitions.csv` and `individuals.csv`.
- `docker/postgres-init/01-create-odds.sql`: creates the auxiliary `odds`
  database used by `ODDS_DATABASE_URL`.

Generated finalized CSVs, model artifacts, prediction outputs, logs, screenshots,
and database dumps stay out of git.

## Commands

Use `uv` as the Python command runner.

- First-time setup: `setup.ps1` on Windows or `./setup.sh` on macOS/Linux.
- Local web app: `uv run mma-web`.
- Docker app: `docker compose up --build`.
- Scrape raw UFCStats CSVs: `uv run mma-scrape-ufcstats`.
- Rebuild generated schemas and finalized CSVs: `uv run mma-rebuild-db --reset-db`.
- Normal public data update after initial Hugging Face import:
  `uv run mma-rebuild-db --scrape --reset-db --odds-features`.
- Train model from CLI: `uv run mma-train`.
- Evaluate model artifacts: `uv run mma-evaluate --write-report --format text`.
- Predict from CLI: `uv run mma-predict`.
- Release audit: `uv run mma-release-audit`.
- Docker smoke: `uv run mma-docker-smoke`.

For tests, prefer the narrowest relevant command first. Common examples:

- Web/API/docs: `uv run pytest tests/test_web -q`.
- Release docs only: `uv run pytest tests/test_web/test_release_docs.py -q`.
- Feature calculators: `uv run pytest tests/tests_layer1 tests/tests_layer2 tests/tests_layer3 -q`.
- Inference: `uv run pytest tests/test_inference -q`.
- Full suite: `uv run pytest`.

## Runtime Rules

- Importing the web app must be light and side-effect free. Do not import
  AutoGluon, start Scrapy, connect to Postgres, call Wikipedia, call LLMs, or
  hit external APIs at import time.
- Long-running Data and Predict actions must run as background jobs. Preserve
  stdout, stderr, subprocess command lines, tracebacks, and script output in
  `data/logs/jobs`; full logs are exposed at `/api/jobs/{job_id}/log`.
- Dashboard jobs are serialized. Do not add concurrent writers for model/data
  artifacts without explicit design work.
- Docker Compose must keep Postgres at `postgres:18.1` and mount
  `docker/postgres-init/01-create-odds.sql` so the `odds` database exists.
- The setup scripts download from the Hugging Face dataset, verify checksums,
  restore both database dumps, copy processed CSVs, extract the starter
  `ag-20260304_110750-win-extreme` model, optionally write LLM configuration,
  and start the dashboard.

## Dashboard Surface

Data tab:

- Incrementally scrape `competitions.csv` and `individuals.csv`.
- Rebuild the PostgreSQL feature store.
- Recalculate odds features from the imported Hugging Face `odds` database.
- Write `prediction_data.csv`, `training_data.csv`, and
  `training_data_dec.csv`.
- Run read-only analytics. Live BestFightOdds refresh is opt-in and not part of
  the default dashboard update.

Predict tab:

- Select a model from `MMA_AI_MODELS_DIR`, defaulting to `AutogluonModels`.
- Load upcoming events from Wikipedia into an event-name dropdown.
- Predict events and manual fighter matchups through the same `predict.py` and
  `InferenceDataBuilder` path.
- Prediction-time live/manual odds are enabled by default but are used for
  market probability, expected value, and pick edge reporting. Historical odds
  columns may exist in generated CSVs and model artifacts; inspect `feats.txt`
  before claiming whether a specific model used odds.
- Manual matchup odds controls are not exposed in the dashboard.
- The advanced Flaresolverr proxy toggle is only for BestFightOdds blocking
  normal odds scraping.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanMcInerney/mma-ai](https://github.com/DanMcInerney/mma-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
