---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup
uv venv .venv --python 3.11 && source .venv/bin/activate && uv pip install -e ".[dev]"

# Full CI suite (must all pass)
ruff check . && ruff format --check . && mypy src/ && pytest

# Individual checks
ruff check .                # lint
ruff format .               # auto-format
mypy src/                   # type-check (strict mode, src/ only — mcp module excluded)
pytest                      # all tests
pytest tests/test_oauth.py  # single test file
pytest -k "test_name"       # single test by name

# Run / inspect the server
hermes-mcp serve            # or: python -m hermes_mcp serve
hermes-mcp doctor           # startup self-check (probes the gateway)
hermes-mcp mint-client      # generate a fresh OAuth client_id / client_secret
```

## Architecture

**hermes-mcp** is an MCP bridge that lets Claude Desktop / Claude.ai delegate tasks to a locally running **Hermes Agent**. Claude calls one MCP tool (`hermes_ask`) over an HTTPS tunnel; the bridge gates that with OAuth 2.1 and forwards each call to the Hermes gateway's OpenAI-compatible HTTP API.

```
Claude.ai
  │  HTTPS via cloudflared tunnel
  ▼
hermes-mcp (this project, listening on 127.0.0.1:8765)
  ├─ OAuth 2.1 (authorization code + PKCE), single static client_id/secret
  └─ HTTP POST to the gateway
     │
     ▼
hermes-gateway (127.0.0.1:8642, OpenAI-compatible /v1/chat/completions)
  └─ same AIAgent loop that drives Telegram (skills, tools, sessions)
```

The gateway is a **separate, long-running process** owned by the user (typically a `systemd --user` service). hermes-mcp does not spawn it; it just sends HTTP requests.

The six source modules in `src/hermes_mcp/` have clean single responsibilities:

- **`config.py`** — frozen `Config` dataclass parsed from env vars. Required: `OAUTH_CLIENT_ID`, `OAUTH_CLIENT_SECRET`, `OAUTH_ISSUER_URL`, `HERMES_API_KEY`. Validates the issuer URL is HTTPS (or `http://localhost`), the client_secret is ≥32 chars, and warns if `BIND_HOST` is non-loopback.
- **`oauth.py`** — `StaticClientProvider` implements the MCP SDK's `OAuthAuthorizationServerProvider` protocol with one pre-shared client. Mints opaque 256-bit access tokens (1h TTL) and refresh tokens (30d, rotated atomically on use). PKCE-S256 enforced by the SDK. DCR is disabled. `_StaticClient.validate_redirect_uri` enforces a scheme allowlist (`https`, `http`-on-localhost, `claude`, `claudeai`) so `/authorize` cannot become an open redirector to `javascript:` / `data:` URIs.
- **`hermes_client.py`** — `HermesClient.ask()` does `httpx.post` to the gateway's `/v1/chat/completions` with `Authorization: Bearer $HERMES_API_KEY`. `session_id` is forwarded as `X-Hermes-Session-Id`. `toolsets` is accepted for backward-compat but ignored — toolset selection now lives in the Hermes config (`platform_toolsets.api_server`). Gateway error bodies are NOT echoed in user-visible errors (DEBUG only).
- **`jobs.py`** — `JobStore` is a thread-safe in-memory dict of `Job` records, used by `hermes_ask(..., async_mode=True)`, `hermes_check`, `hermes_cancel`, and `hermes_reset`. Lazy TTL reap (24h) on every access, 1000-job cap. In-memory only by design; restart drops everything. `mark_completed`/`mark_failed` are terminal-state-aware so a late-finishing worker cannot overwrite a cancellation. `reset_all()` reaps first, then wipes the store and returns `(cleared, by_status)`. Times use `time.time()` (wall clock, epoch seconds) so they round-trip cleanly through JSON to the caller; small risk of confusion if the system clock jumps backwards, accepted in exchange for code simplicity.
- **`server.py`** — `build_app()` constructs a `FastMCP` instance with `auth_server_provider`, `AuthSettings`, and `transport_security`. Registers four tools: `hermes_ask` (sync default; `async_mode=True` spawns a daemon thread and returns a `job_id`), `hermes_check(job_id)`, `hermes_cancel(job_id)`, and `hermes_reset()`. FastMCP itself adds `/authorize`, `/token`, `/.well-known/oauth-authorization-server`, and the `RequireAuthMiddleware` that gates `/mcp`. `serve()` runs uvicorn.
- **`doctor.py`** — `run_checks()` probes the gateway's `/v1/health` (no auth) and `/v1/models` (with `HERMES_API_KEY`); warns if `HERMES_MODEL` isn't in the returned model list.

**Four-tool design.** The tools form a tight lifecycle: submit (`hermes_ask`), poll (`hermes_check`), abandon a single job (`hermes_cancel`), wipe the store (`hermes_reset`). Do not add tools for *new* use cases (different actions, different domains) without discussing in an issue first.

**`hermes_reset` is a global operation.** The job store is shared across every MCP caller (multiple Claude sessions, background Hermes-agent workflows, etc.). Resetting wipes them all. The tool description warns the LLM to confirm with the user before calling it when other work might be in flight.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlennie/hermes-mcp](https://github.com/mlennie/hermes-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
