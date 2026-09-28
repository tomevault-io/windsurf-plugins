---
trigger: always_on
description: This file provides guidance to CodeBuddy Code when working with code in this repository.
---

# CODEBUDDY.md

This file provides guidance to CodeBuddy Code when working with code in this repository.

## Commands

All targets run through `uv` and the Makefile:

```bash
make all       # ship bar: format + lint + typecheck + test — what pre-commit runs
make format    # ruff check --fix + ruff format on src/ tests/ (rewrites files)
make lint      # ruff check + ruff format --check on src/ tests/
make typecheck # mypy strict on src/
make test      # full suite with coverage (uv run pytest --cov=octop_browser)
make build     # uv build → wheel + sdist
make mcp       # launch the MCP stdio server (python -m octop_browser.mcp_server)
make publish   # local uv publish (prefer /publish skill + Actions; do not use for public releases)
make clean     # remove dist/, .coverage, .mypy_cache/, .ruff_cache/
```

**Git hooks (required before committing):** run **`make install-hooks`** once per clone. It sets `core.hooksPath=.githooks`, so every `git commit` first runs **`make all`** (`format` rewrites files, then lint / typecheck / test). Staged files rewritten by `format` are re-added automatically, so the commit contains the formatted content. This replaces the `pre-commit install` route — `core.hooksPath` overrides `.git/hooks/`, and `make all` is a superset of `.pre-commit-config.yaml` (ruff + ruff-format + mypy, plus tests). Bypass only in emergencies: `SKIP_PRECOMMIT=1 git commit …` or `git commit --no-verify`; never skip hooks to land a red suite.

Useful narrower invocations:

```bash
# Unit tests only — no Chrome required
uv run pytest tests/unit/ -q

# A single test file or test
uv run pytest tests/unit/test_mode.py -q
uv run pytest tests/unit/test_settings.py::test_env_override_browser_mode -q

# Integration tests — auto-skipped when Chrome/Chromium is not on the host
uv run pytest tests/integration/ -v

# Type-check or lint just one path
uv run mypy src/octop_browser/session.py
uv run ruff check src/octop_browser/cdp/

# Install dev extras + enable the commit gate
uv sync --extra dev
make install-hooks
```

`pytest-asyncio` runs in `asyncio_mode = "auto"` (set in `pyproject.toml`) — async test functions need no decorator. `tests/integration/conftest.py` skips the whole integration suite when `find_chrome()` returns None, so unit work on machines without Chrome stays green.

## Architecture

This package wraps Chrome over **pure CDP** (WebSocket, no Playwright) and exposes it three ways: an async Python API, a stateless `browser_tool()` for AI frameworks, and an MCP server. Understanding the call chain matters when changing anything in the launch / connect / DOM path.

### Layered call chain

```
browser_tool()  ──► BrowserSession ──► _InternalCDPSession ──► CDPClient (websockets) ──► Chrome
mcp_server.py   ──┘                          │
                                             └─► launcher.launch_or_attach + get_page_ws_url
```

- `tool_interface.browser_tool(action, profile=..., mode=...)` — stateless dispatch keyed by `profile`. Maintains a `_registry: dict[str, BrowserSession]`; first call per profile creates a session, subsequent calls reuse it. Action names map directly to `BrowserSession` methods via `getattr`.
- `session.BrowserSession` — public async API. Mixes in `HooksMixin` (events: `before_action`, `after_action`, `action_error`; `page_navigated` is reserved and never fired) and accumulates running totals in `metrics_summary()`.
- `session._InternalCDPSession` — owns one `CDPClient` + one `RefCache` per page. Honors `cfg.cdp_ws_url`: if set, **bypasses the launcher entirely** and connects directly to the given WebSocket (used for remote/Docker Chrome).
- `cdp.client.CDPClient` — minimal CDP framing over `websockets.asyncio.client`. `send(method, params)` returns the result; `enable_domain()` enables `Page` / `DOM` / `Runtime` / `Input` after connect.
- `cdp.launcher` — `find_chrome()`, `launch_or_attach()` (re-attach if port is already serving CDP), `_get_ws_url()` polls `http://{cdp_host}:{port}/json/version`, `get_page_ws_url()` picks the first `type==page` target. All HTTP probes go through `aiohttp` and **always read `cdp_host` from settings** — never hardcode `localhost`.

### Configuration: OctopSettings (single source of truth)

`settings.OctopSettings` is a `dataclass` whose every field uses `field(default_factory=lambda: _env_*(...))`. This means **each instantiation re-reads env vars**, which is what `monkeypatch.setenv` in tests relies on. The module-level `settings` singleton (read at import time) is the default everywhere; tests should construct a fresh `OctopSettings()` rather than mutating it.

Env vars (all optional):

| Variable | Default | Used in |
|---|---|---|
| `BROWSER_USE_PROFILES_DIR` | `~/.octop-browser/profiles` | `ProfileManager.__init__` |
| `BROWSER_USE_CDP_HOST` | `localhost` | `launcher._get_ws_url`, `_port_in_use`, `get_page_ws_url`, `session.list_tabs`; also drives `--remote-debugging-address` when non-loopback |
| `BROWSER_USE_CDP_PORT_START` | `9222` | `ProfileManager` port assignment |
| `BROWSER_USE_MODE` | `auto` | resolved via `mode.resolve_headless()` |
| `BROWSER_USE_CHROME_BIN` | auto-detect | `launcher.find_chrome` (checked before path search) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TencentCloud/octop-browser](https://github.com/TencentCloud/octop-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
