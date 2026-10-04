---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code style

- Code must be easy for a human to understand first: plain naming, straightforward control flow, standard idioms for the language and framework. No cleverness that needs a comment to decode.
- If less code solves the problem equally well, prefer less code — but never buy brevity at the cost of readability.

## Repository Layout

This is a polyglot monorepo with **four independent backend services** that share PostgreSQL/Redis/Kafka infrastructure, plus a separate frontend SPA. Each service has its own dependencies and build:

- **Root (`src/`)** — RDA (Real-Time Detection Agent), TypeScript/Fastify HTTP API. Owns the `package.json` at the repo root, the Knex migrations under `src/database/migrations/`, and the shared ONNX model under `models/`.
- **`paa-service/`** — PAA (Pattern Analysis Agent), TypeScript Kafka consumer worker. Has its own `package.json`, `tsconfig.json`, and `node_modules`. Not invoked from the root scripts.
- **`mla-service/`** — MLA (Model Learning Agent), Python 3.11 service. Has its own `requirements.txt` and `venv`. Trains XGBoost → ONNX models that are deployed by copying into `models/fraud_model.onnx` for RDA.
- **`fia-service/`** — FIA (Fraud Investigation Agent), Python 3.11 service. Consumes the `transactions.blocked` Kafka topic, runs a fine-tuned Phi-3-mini-4k-instruct LLM, and writes structured reports to PostgreSQL `investigationReports`. Strictly async — never on the RDA authorization path.
- **`frontend/`** — Sentinel operator dashboard, Vite + React 18 SPA. Has its own `package.json` and `node_modules`. Not invoked from the root scripts. Talks to RDA `/v1/admin/*` and FIA `/v1/reports*`; when those services are unreachable every read returns an empty fallback and the SPA shows a persistent OFFLINE banner — no synthetic data. The old `mock.js` seed and `sentinel.useMock` override were removed in May 2026.

RDA is the only producer. PAA and MLA consume `transactions.completed`; FIA consumes `transactions.blocked` (published by RDA only when the decision is `DECLINE`). All four services share the same Postgres `fraud_db`.

## Common Commands

### RDA (root)
```bash
npm run start:dev          # nodemon hot-reload
npm run build              # tsc; postbuild copies *.yaml into dist/
npm run lint               # eslint over .ts
npm run test               # jest --runInBand --passWithNoTests
npx jest path/to/file.test.ts   # single test file
npx jest -t "test name"         # single test by name
npm run db:migrate         # knex migrate:latest (uses dotenv)
npm run db:migrate:make -- name_of_migration
npm run db:migrate:rollback
```

### PAA (`paa-service/`)
PAA has its own deps and tsconfig — `cd paa-service` first. Same `start:dev` / `build` / `lint` / `test` script names. The root `npm install` does **not** install PAA deps.

### FIA (`fia-service/`)
```bash
cd fia-service && source .venv/bin/activate     # NOTE: .venv (FIA), not venv (MLA)
python -m src.main                              # consume + generate reports
```
Health: `:9094/livez`, `:9094/readyz`, `:9094/stats`. The first run downloads ~7.6 GB of Phi-3-mini-4k-instruct weights to `~/.cache/huggingface`. Set `FIA_FALLBACK_ON_LLM_FAILURE=true` (default) to degrade gracefully to a deterministic rule-based report when the LLM cannot load — the pipeline still produces parseable rows. Device selection: `LLM_DEVICE=auto` picks CUDA → MPS → CPU; on Apple Silicon expect ~45 s model load and a one-time ~6–10 min MPS kernel compilation on the first generation, then ~40–90 s per report steady-state.

### MLA (`mla-service/`)
```bash
cd mla-service && source venv/bin/activate
python -m src.main                          # start drift-monitoring loop
python -m src.main --train                  # force retrain on startup
python scripts/train_initial_model.py       # cold-start training
python scripts/train_initial_model.py --samples 10000 --skip-registry
pytest                                      # all tests
pytest tests/test_onnx_conversion.py -v     # single file
```
After training, deploy the model to RDA with `cp models/fraud_model_v1.0.onnx ../models/fraud_model.onnx`.

### Frontend (`frontend/`)
The Sentinel dashboard has its own deps and build — `cd frontend` first. The root `npm install` does **not** install frontend deps.
```bash
cd frontend && npm install
npm run dev                                  # vite dev server on :5173
npm run build                                # production bundle into dist/
npm test                                     # vitest run (jsdom + Testing Library)
npm test -- tests/sidebar.test.jsx           # single file
```
The dev server proxies `/v1/*` → `VITE_RDA_URL` (default `http://localhost:3000`) and `/fia/*` → `VITE_FIA_URL` (default `http://localhost:9094`, prefix stripped). Auth is read from `localStorage` at request time — `sentinel.jwt` becomes `Authorization: Bearer …` (the user JWT from `POST /v1/auth/login`), `sentinel.apiKey` becomes `X-Api-Key`. See [`docs/FRONTEND.md`](docs/FRONTEND.md) for the full reference.

### Infrastructure
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ojuri-io/ojuri](https://github.com/ojuri-io/ojuri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
