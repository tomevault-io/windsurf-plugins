---
trigger: always_on
description: poetry install                        # install deps
---

# CC-Adapter — Agent Guide

## Commands

```bash
poetry install                        # install deps
poetry run pytest                     # run full test suite
poetry run pytest tests/test_tool_mapping.py -v  # single test file
poetry run black .                    # format (line-length 120)
poetry run python -m cc_adapter       # dev server (port 8080, or $CC_ADAPTER_PORT)
poetry run cc-adapter                 # same, via pyproject.toml scripts
bash run.sh                           # alternative: sources .env, runs uvicorn directly
docker build -t dgqyushen/command-code-proxy:latest .
docker compose up -d                  # docker-compose.yml + optional docker-compose.override.yml
```

**Version**: `pyproject.toml` `[tool.poetry].version` is the single source of truth. `core/constants.py` reads it at import via `_load_version()`. Both `main.py` and `admin/router.py` import `VERSION` from constants. Bump in pyproject.toml when releasing — do not edit constants.py.

**After each feature/fix**: bump the version in `pyproject.toml` before committing. Push a `v*` tag (e.g. `v0.7.2`) to trigger the Docker CI workflow, which pushes `latest` + semver tags to Docker Hub.

## Routes

| Path | Handler | Auth |
|---|---|---|
| `GET /` | `main.py` | none (redirects to /admin/) |
| `GET /health` | `main.py` | none |
| `POST /v1/chat/completions` | `providers/openai/router.py` | access_key |
| `POST /v1/messages` | `providers/anthropic/router.py` | access_key (or x-api-key) |
| `POST /v1/responses` | `providers/openai/responses_router.py` | access_key |
| `GET /v1/models` | `main.py` (dynamic via `get_models_data()`) | none |
| `GET /admin/api/models` | `admin/router.py` (public listing, no auth) | none |
| `POST /admin/api/models/refresh` | `admin/router.py` | admin auth |

Entry: `cc_adapter/__main__.py` → `main.py:run()` → uvicorn. Import: `from cc_adapter.main import app`.

**Module-level side effect**: `main.py:58` calls `cfg = AppConfig()` and `runtime_init(cfg, create_client(cfg))` at import time. Tests and any code that imports `cc_adapter.main` must be aware of this — it creates a real client with `.env` defaults.

**`docs/` is gitignored**: Documentation lives outside the repo. Do not create files under `docs/`.

## Config (prefix `CC_ADAPTER_`)

Fields in `core/config.py:AppConfig` (loaded eagerly from `.env` at import).

| Env var | Default | Notes |
|---|---|---|
| `CC_ADAPTER_CC_API_KEY` | — | `str \| list[str]` — JSON array: `["k1","k2"]` |
| `CC_ADAPTER_ACCESS_KEY` | — | Bearer token auth (all endpoints) |
| `CC_ADAPTER_CC_BASE_URL` | `https://api.commandcode.ai` | |
| `CC_ADAPTER_DEFAULT_MODEL` | `deepseek/deepseek-v4-flash` | |
| `CC_ADAPTER_HOST` | `0.0.0.0` | |
| `CC_ADAPTER_PORT` | `8080` | |
| `CC_ADAPTER_LOG_LEVEL` | `INFO` | |
| `CC_ADAPTER_LOG_FORMAT` | `console` | or `json` |
| `CC_ADAPTER_ADMIN_PASSWORD` | — | Admin panel password |
| `CC_ADAPTER_HTTP_MAX_CONNECTIONS` | `200` | |
| `CC_ADAPTER_HTTP_MAX_KEEPALIVE_CONNECTIONS` | `50` | |
| `CC_ADAPTER_HTTP2` | `false` | |
| `CC_ADAPTER_ZDR` | `true` | Sends `x-cmd-zdr: 1` header (zero data retention) |
| `CC_ADAPTER_OSS_PRIMARY_PROVIDER` | — | Optional OSS provider name, sent as `x-oss-primary-provider` header |

## Architecture

```
POST /v1/messages → providers/anthropic/request.py → command_code/client.py
POST /v1/chat/completions → providers/openai/request.py → command_code/client.py
Both translate to CC /alpha/generate body, stream SSE back.
```

- **Two translator pairs** in `providers/anthropic/` and `providers/openai/` (request→CC, response←CC); shared code in `providers/shared/`, `command_code/`, `core/`.
- **Singletons** owned by `core/runtime.py`: `_config`, `_cc_client`, translator instances (lazy init via `get_*()`). Also `_version_checker` and `_model_fetcher`.
- **`get_or_create_client()`** at `runtime.py:41` — auto-creates a client with `AppConfig()` defaults if `init()` hasn't been called, logging a warning. Used by all routers when no client is available.
- **Auth headers**: `core/headers.py` — `extract_token()` (Bearer/x-api-key), `auth_error_response(message, protocol)` (401). Branches on `protocol: "openai" | "anthropic"` for correct error shape; `message` parameter allows custom error text.
- **Retry**: `core/retry.py` — `retry_on_empty()` for non-streaming (retries once on empty upstream response), `stream_with_retry()` for streaming (same retry logic + optional error event emission).
- **Admin auth**: HMAC-signed token in `core/auth.py` (not JWT); embeds `exp` + password hash prefix. API access validation at `core/auth.py:check_api_access()`.
- **ID generation**: `generate_id(prefix, length)` in `core/utils.py`.
- **Constants**: `core/constants.py` — `STREAMING_HEADERS`, `NPM_URL`, `NPM_CACHE_TTL`, `NPM_ERROR_BACKOFF`, `KEY_CREDITS_CACHE_TTL`, `KEY_CREDITS_ERROR_BACKOFF`, `VERSION`.
- **Version checker**: Background npm polling, cached 30min, fallback `0.25.2` (env `CC_ADAPTER_DEFAULT_VERSION`). See `core/version_checker.py`. Tests must set `_last_fetch_time = None` (not `0.0`) to guarantee cache invalidation.
- **Model fetcher**: `core/model_fetcher.py` — fetches models/reasoning-efforts from CC API, feeds into `MODEL_PROVIDER_MAP` and `MODEL_REASONING_EFFORTS_MAP` via `refresh_maps()`.

### CC Request Headers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dgqyushen/command-code-adapter](https://github.com/dgqyushen/command-code-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
