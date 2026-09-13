---
trigger: always_on
description: source .venv/bin/activate
---

# TalkWithMe — Agent Instructions

## Run the app

```bash
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
deactivate
```

Requires a locally running **llama.cpp** server with an OpenAI-compatible API (or any OpenAI-compatible LLM, local or remote). TTS and STT servers are optional.
Open `http://localhost:8000` in a browser.

A remote LLM that needs an API key is supported: set the `TALKWITHME_LLM_API_KEY` env var or create an `llm_api_key` file in the project root (copy `llm_api_key.example`) — see the README "LLM API key (remote LLMs)" section. The key is never in `settings.yaml`, never in the UI, and never logged.

## Testing — pytest, fully offline

```bash
source .venv/bin/activate
pip install -r requirements-dev.txt
python -m pytest # config in pytest.ini
deactivate
```

- **No servers needed.** The suite is hermetic: every external HTTP endpoint (LLM, TTS, STT, MCP) is faked via `tests/factories.py` (fake `httpx.AsyncClient`s, config factories, SSE helpers). It must run — and pass — with nothing but Python installed.
- **Isolation**: `tests/conftest.py` has an autouse fixture that points every module-level global at per-test `tmp_path` state: the config caches in `app/config.py`, `_PERSISTENCE_ROOT` (in both `app/persistence.py` and `app/routers/persistence.py` — the router imported it *by value*, so it needs its own patch), the `session` singleton, the MCP tool registry, the TTS capabilities cache in `app/services/tts_client.py`, the LLM API key cache in `app/services/llm_auth.py` (its `_PROJECT_ROOT` is re-pointed to `tmp_path` and `TALKWITHME_LLM_API_KEY` is deleted from the environment, so a test's key — or the developer's real one — never leaks into another test), and the once-per-URL cleartext-warning dedupe set in `app/services/llm.py`. Real `settings.yaml` / `Personas/` / `chatrooms.yaml` / `chatrooms/` / `llm_api_key` data is never read or written. **If you add a new module-level global to the app, add it to that fixture.**
- **Lifespan**: the `client` fixture uses `TestClient` *without* the startup lifespan, because the lifespan re-reads the real YAML files (clobbering test caches) and attempts MCP discovery. Tests that exercise the lifespan do so explicitly with a local `TestClient` in a `with` block and monkeypatched `load_*`/`load_tools` (see `tests/test_main.py`).
- **Coverage map**: `test_config.py` (config models + YAML load/save), `test_persona_store.py` (persona directory discovery: frontmatter, file ops, language/avatar helpers, memory-file append/purge, YAML→dir migration), `test_builtin.py` (the built-in `add_memory` tool: registry, availability gating, save/error paths), `test_models.py` (API request/response models), `test_persistence.py` (disk persistence + audio staging), `test_session_manager.py`, `test_llm.py` (SSE parsing + agentic tool loop + the LLM API key header + the `http://` cleartext warning), `test_llm_auth.py` (LLM API key resolution: env-var/file priority, the `llm_api_key = <value>` file format, lazy load, the never-logged invariant), `test_mcp_client.py`, `test_tool_registry.py`, `test_tts_stt_clients.py`, `test_tts_fixtures.py` (the two copies of the four capabilities snapshots — pytest's `tests/factories.py` vs the plain-Node `tests/fixtures/*.json` — must not drift; re-copy both from tts-serve in the same commit), `test_chat_sse.py` (the `/api/chat` SSE endpoint — LLM stubbed, selection/persistence/echo/tool events for real), `test_main.py`, `test_docs.py` (the AGENTS.md API endpoints table must match the routes actually registered on the app — run it after adding/removing endpoints and update the table), one `test_routers_*.py` per API router, `test_persona_form.js` (plain Node, **not** part of pytest — see below: the persona editor form logic in `static/persona.js`), and `test_tts_settings.js` (plain Node, **not** part of pytest: the dynamic TTS parameter section in `static/tts-params.js` + the TTS wiring in `static/settings.js`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scorbo2/TalkWithMe](https://github.com/scorbo2/TalkWithMe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
