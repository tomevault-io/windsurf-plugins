---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

MinerU Tianshu (天枢) — an AI data preprocessing platform that turns PDFs, Office docs, images, audio,
video and bioinformatics files into Markdown + structured JSON. Vue 3 frontend, FastAPI API server,
LitServe GPU worker pool, SQLite task queue, optional Redis queue / MCP server / RustFS object storage.

Code comments, logs and docs are predominantly Simplified Chinese — match that when editing existing files.

## Commands

### Docker (primary deployment path)

```bash
make setup           # first-time: copy .env, create dirs, build, start
make build | start | stop | restart | status | logs
make logs-worker     # per-service logs (also logs-backend, logs-frontend)
make shell-worker    # exec into a container
make test-gpu        # verify torch/paddle CUDA inside the worker
make validate        # docker compose config
make dev             # docker-compose.dev.yml (hot reload + debugpy)
```

`make` reads `REDIS_QUEUE_ENABLED` from the root `.env` and adds `--profile redis` when true.
Offline/air-gapped builds: `scripts/build-offline.sh`, then `deploy-offline.sh` (or `deploy-offline-cpu.sh`).

### Local backend

```bash
cd backend
cp .env.example .env          # REQUIRED — start_all.py exits if backend/.env is missing
python start_all.py                       # API 8000 + worker 8001
python start_all.py --enable-mcp --mcp-port 8002
python start_all.py --workers-per-device 2 --devices 0,1 --accelerator cuda
```

All backend processes must run with `backend/` as CWD (imports are top-level: `from utils import ...`,
`from auth import ...`). Individual services: `python api_server.py`, `python litserve_worker.py`,
`python task_scheduler.py --enable-scheduler`, `python mcp_server.py`.

### Frontend

```bash
cd frontend && npm install
npm run dev      # :3000, proxies /api -> localhost:8000 (vite.config.ts)
npm run build    # tsc && vite build -> dist/
```

### Lint

There is **no test suite**. The quality gate is pre-commit, which is also the only CI job
(`.github/workflows/pylint.yml`):

```bash
pip install pre-commit && pre-commit install
pre-commit run --all-files
ruff format backend/ && ruff check --fix backend/
```

Ruff: `select = ["E","F"]`, `ignore = ["E402","E501"]`, line-length 120, target py312, double quotes.
Hooks also run shellcheck, markdownlint, and enforce LF endings (except `.bat/.cmd/.ps1`).

## Architecture

### Process topology

`start_all.py` spawns independent subprocesses; in Docker each is a separate container running the same
`tianshu-backend` image with a role argument via `scripts/docker-entrypoint.sh` (`api` / `worker` / `mcp`):

| Service | Default port | Entry |
|---|---|---|
| API server (FastAPI) | 8000 | `api_server.py` |
| Worker pool (LitServe) | 8001 | `litserve_worker.py` |
| MCP server | 8002 | `mcp_server.py` |
| Scheduler (optional) | — | `task_scheduler.py` |
| Frontend (nginx) | 80 | `frontend/` |

### Pull-based task queue

The API **never calls the worker**. It writes a row to the `tasks` table and returns immediately.
Each worker runs `_worker_loop()`, polling `TaskDB.get_next_task()` every `--poll-interval` (0.5s).

Claiming is atomic: `BEGIN IMMEDIATE` plus `UPDATE ... WHERE task_id=? AND status='pending'`, retried when
`rowcount == 0` (another worker won the race). With `REDIS_QUEUE_ENABLED=true`, `redis_queue.py`
(sorted-set priority queue) is tried first and SQLite is the automatic fallback — SQLite stays the source
of truth for metadata and results either way.

`task_db.py` owns the schema and performs **in-place migrations on startup** by `SELECT`ing a column and
catching `sqlite3.OperationalError` to `ALTER TABLE`. Add new columns with that same pattern, never by
editing `CREATE TABLE` alone — existing deployments would not pick it up.

`auth/auth_db.py` reuses the **same SQLite file** (`DATABASE_PATH`) as the task DB.

### Engine routing (`litserve_worker.py::_process_task`)

Per-task pipeline: vLLM container switch → Office conversion → PDF split → watermark removal →
engine dispatch → normalize → persist.

Dispatch is keyed on the task's `backend` string:

- `sensevoice` → audio engine; `video` → video engine
- `paddleocr-vl`, `paddleocr-vl-vllm` → PaddleOCR-VL
- anything containing `pipeline` / `vlm-` / `hybrid-` → MinerU (`options["parse_mode"] = backend`)
- `auto` → sniffed by extension: format engines → audio → video → MinerU → LibreOffice conversion for
  legacy `.doc/.xls/.ppt` → MarkItDown fallback
- otherwise → looked up in `FormatEngineRegistry`

Every engine is imported behind a try/except with an `X_AVAILABLE` flag, so a missing optional dependency
degrades that one backend instead of killing the worker. Preserve that pattern when adding engines.

`VLLMController.ensure_service()` enforces **mutual exclusion between vLLM containers**
(`tianshu-vllm-paddleocr` vs `tianshu-vllm-mineru`) by stopping the conflicting one to free VRAM — the
worker talks to the Docker socket to do this.

### Output contract

Every engine's raw output directory goes through `output_normalizer.normalize_output(dir, handle_method)`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magicyuan876/mineru-tianshu](https://github.com/magicyuan876/mineru-tianshu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
