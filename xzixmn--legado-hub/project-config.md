---
trigger: always_on
description: Repo-specific guidance for OpenCode sessions working in `legado-hub`.
---

# AGENTS.md

Repo-specific guidance for OpenCode sessions working in `legado-hub`.
Keep the root clean; see `docs/architecture/repository-layout.md` for the boundary rules.

## Stack

- **backend/**: FastAPI runtime with a public/reader entrypoint on `8765` and an admin entrypoint on `8766`. The deployment entrypoint is `python -m app.server`; `app.main:app` is the combined compatibility/test app. Run commands from `backend/`.
- **frontend/**: React 19 + Vite + shadcn/ui console. The same built dist is served on both ports, while `/api/auth/entrypoint` selects the reader-only or administrator login/UI surface. Vite proxies to `8765` by default; set `VITE_LEGADOHUB_ENTRYPOINT=admin` to proxy to `8766`.
- **plugins/**: source plugins loaded at runtime by `app/source_plugins/loader.py`, which recursively scans `plugins/sources/**/metadata.yaml`.

## First-time setup / dev run

Docker Compose with the published `xzixmn/legado-hub:latest` image is the recommended deployment path. `start.bat` is the Windows source-development bootstrap: it creates `.venv`, installs `backend/requirements.txt`, runs `python -m playwright install chromium`, builds the frontend, then starts uvicorn. Manual equivalent:

```
python -m venv .venv
.venv/Scripts/python.exe -m pip install -r backend/requirements.txt
.venv/Scripts/python.exe -m playwright install chromium
cd frontend && npm install && npm run build
cd ../backend && ../.venv/Scripts/python.exe -m app.server --host 0.0.0.0 --public-port 8765 --admin-port 8766
```

On first startup the lifespan handler initializes the SQLite DB at `backend/data/app.db`, creates `admin`, and prints a generated high-entropy password once when no password was configured. Explicit passwords are never echoed. The product supports only local or trusted-LAN operation; external tunneling and its network security are operator-owned and do not select an application mode.

## Commands

Backend (run from `backend/`):
- Run server: `python -m app.server --host 0.0.0.0 --public-port 8765 --admin-port 8766`
- Tests: `pytest` (config lives in `backend/pytest.ini`; `asyncio_mode = auto`)
- Single test: `pytest ../dev-assets/tests/test_shared_book_storage.py`
- Maintenance scripts: `python scripts/create_source_plugin.py`, `python scripts/validate_source_plugin.py` (these are the only scripts that belong in `backend/scripts/`; put probes/benchmarks in `dev-assets/`)

Frontend (run from `frontend/`):
- Dev server: `npm run dev`
- Build: `npm run build` (runs `tsc -b` then `vite build`)
- Lint: `npm run lint`
- Tests: `npx vitest` (jsdom; no `test` script defined; config is inside `vite.config.ts` via the `vitest/config` import)

There is no command-order contract beyond: build frontend before running the server in Docker/production, since the backend serves `frontend/dist`.

## Validation cadence

- Do not run the full test suite after every small code edit.
- For a small task, finish the complete scoped change first, then run the relevant tests once as a batch.
- For a large task, split the work into explicit phases and run the phase-relevant tests once at the end of each phase.
- Focused syntax checks or a single regression test are allowed while diagnosing a concrete failure; they do not replace the phase gate.
- Before any commit, push, release, or claim that implementation is ready to commit, run the canonical full verification once with `verify.ps1`.
- Docker, Compose, image, or deployment changes must also be accepted on the SSH host `本地测试` by default; preserve its bind-mounted runtime data and use an isolated temporary container for destructive smoke checks. A missing local Docker CLI is not a reason to skip this gate.
- Never claim a phase or task passed without reporting the actual commands and results from its scheduled validation gate.

## Tests are split between repo and local-only `dev-assets/`

This is the most likely thing to trip you up:

- `dev-assets/` is **gitignored**. Only a small allow-list of test files plus `dev-assets/tests/conftest.py` are committed (see the `!dev-assets/tests/...` block in `.gitignore`).
- `backend/pytest.ini` points `testpaths` to `../dev-assets/tests` and `--ignore`s many files there.
- `dev-assets/tests/conftest.py` *also* `pytest_ignore_collect`s a second set of files (live-acceptance, official-auth, source-plugin-fixture tests, etc.) because those depend on local-only assets.
- Net effect: a fresh checkout runs only the committed subset. Do not assume a test file you see referenced exists in the repo; do not rely on `dev-assets/tests/source_plugins/`, `official_auth/`, or any `test_*` listed in the ignore blocks.
- `dev_assets_test_loader.py` at the repo root contains a **hardcoded absolute path** `C:\Home\Workspace\UGit\legado-hub\...`; it breaks if the repo is relocated. Prefer importing test helpers through normal `pytest`/`pythonpath` rather than extending that loader.

## Plugin contract

- Each plugin is a directory under `plugins/sources/` (subdirs `official/`, `thirdparty/` are scanned recursively) containing `metadata.yaml` + `source.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XziXmn/legado-hub](https://github.com/XziXmn/legado-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
