---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

DocBox is a native desktop app (Windows, macOS on Apple Silicon, Linux) for discovering, downloading, and
running open-source OCR models locally — no login, no cloud calls by default. It checks
device RAM/CPU/disk against each model's requirements before recommending it.

Three-part architecture, distributed as installers via GitHub Releases:

- **Frontend**: Tauri (Rust) native shell (`src-tauri/`) hosting a React + TypeScript +
  Tailwind CSS UI (`frontend/`).
- **Backend**: a local FastAPI + uvicorn server (`src/docbox/backend/`), spawned by the
  Tauri shell as a child process (a venv's `python -m docbox.backend.main`) and killed
  with its whole process tree on exit. The webview talks to it over
  `http://127.0.0.1:<port>`; the frontend polls the `backend_status` Tauri command until
  it's up, then gets the URL from `backend_base_url` (`src-tauri/src/main.rs`).
- **Models**: registered against a `ModelSpec`/`OCREngine` abstraction
  (`src/docbox/backend/core/registry.py`, `src/docbox/backend/engines/`), stored under
  the data dir from `core/paths.py` (`DOCBOX_DATA_DIR`, else platformdirs' user *data*
  dir; never the cache dir).

See "Packaging and the managed runtime" below for how installed builds get Python.

## Commands

### Setup

```sh
uv python pin 3.12      # already pinned via .python-version
uv sync --extra paddle  # base deps + PaddleOCR; --all-extras adds EasyOCR/PyTorch
npm --prefix frontend install
cargo install tauri-cli --version "^2" --locked   # once, for `cargo tauri`
```

### Run the full app

```sh
cargo tauri dev   # from the repository root
```

Run the Tauri CLI from the repository root. With no `package.json` there, it searches
below for one and takes `frontend/` as the frontend directory, which is where
`beforeDevCommand`/`beforeBuildCommand` in `src-tauri/tauri.conf.json` run (so they're
plain `npm run dev`/`npm run build`). Run from `src-tauri/`, it finds no `package.json` and
falls back to the repository root, where those hooks fail. tauri-action in the release
workflow runs from the root too. Dev builds run the repo's `.venv/…/python`, so `uv sync`
must have been run first. A plain `uv sync` is exact: it uninstalls extras you didn't
pass, so always pass the extras you need.

### Build an installer locally

```sh
cargo tauri build   # from the root; needs TAURI_SIGNING_PRIVATE_KEY (updater artifacts)
```

`build.rs` copies `uv` from PATH into `src-tauri/binaries/uv-<target-triple>` when the
sidecar is missing (gitignored); release CI downloads a pinned, checksum-verified uv
there instead.

### Backend only

```sh
uv run python -m docbox.backend.main --port 8756
```

Swagger UI at `http://127.0.0.1:8756/docs`. Useful for iterating on routes without
rebuilding the Rust shell or frontend.

### Frontend only

```sh
npm --prefix frontend run dev     # http://localhost:1420
npm --prefix frontend run build   # production build, also a good typecheck+bundle smoke test
npx tsc --noEmit                  # (run from frontend/) typecheck only
```

Outside the Tauri webview there's no `backend_base_url` command, so `frontend/src/lib/api.ts`
calls `/api` on the page's own origin and Vite's dev/preview server proxies it to
`http://127.0.0.1:8756` (override with `DOCBOX_BACKEND_URL`) — start the backend standalone
alongside it. Same-origin, so the backend's CORS allowlist doesn't apply there.

### Tests and lint

```sh
uv run pytest                                          # full backend suite
uv run pytest tests/backend/test_models_routes.py       # one file
uv run pytest tests/backend/test_platforms.py::test_ollama_status_unavailable_when_not_running  # one test
uv run ruff check src/docbox                             # lint (line-length 100)
```

Backend routes are tested directly via `fastapi.testclient.TestClient(create_app())` —
no subprocess needed. Tests that depend on environment state (a model already
downloaded, `tesseract` on PATH, `easyocr` importable, Ollama running, an NVIDIA key
set) use `pytest.skip`/fixtures rather than mocking those dependencies away — see
`tests/backend/test_models_routes.py::test_download_already_downloaded_model_reaches_done`
and `tests/backend/test_models_catalog.py` for the pattern.

There is no frontend test suite; `npm run build` / `tsc --noEmit` is the verification
step for frontend changes.

## Architecture

### The model registry is the extensibility point

`src/docbox/backend/core/registry.py` defines:

- `ModelSpec` — id, name, engine, description, languages, approx download/RAM/disk, and
  an `engine_factory: Callable[[], OCREngine]`.
- `ModelRegistry` — register/get/list, a plain in-memory dict, module-level `registry`
  singleton.
- `check_fit(spec, caps) -> FitResult` — compares a spec's requirements against
  `DeviceCapabilities` with a RAM safety margin (`_RAM_SAFETY_MARGIN_MB`).
- `recommend(specs, caps)` — Setup's "Recommended start" (`ModelInfo.recommended`): the
  highest-`quality` spec that `runs_smoothly` here, meaning the computer clears its `tier`
  (`_TIER_NEEDS`: total RAM and physical cores), `check_fit` passes and it isn't slow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docboxai/docbox-app](https://github.com/docboxai/docbox-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
