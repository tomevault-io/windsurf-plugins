---
trigger: always_on
description: Python MCP (Model Context Protocol) server that wraps Kanboard's JSON-RPC API. Distributed via `uvx` from GitHub.
---

# mcp-kanboard

Python MCP (Model Context Protocol) server that wraps Kanboard's JSON-RPC API. Distributed via `uvx` from GitHub.

## What this is

- ~55 tools (prefix `kb_`) covering projects, tasks, comments, subtasks, users, categories, tags, search, time tracking, file attachments.
- Single active Kanboard instance per process. Switch local↔remote by changing env vars and restarting the MCP host (Claude Code, MCP Inspector, etc.).
- Destructive tools (`kb_delete_*`, `kb_remove_user`) require explicit `confirm=true` — they raise `ConfirmRequiredError` otherwise.

## Stack

- Python 3.11+ (3.13 dev), `uv` for deps and entry points
- `mcp[cli]>=1.2.0` (FastMCP decorator style)
- `httpx` sync client with Basic auth, 2× retry on transient network errors only
- `pydantic` (schema gen from type hints), `python-dotenv` (local `.env` for dev)

## File map

```
src/mcp_kanboard/
├── __main__.py          # python -m mcp_kanboard → server.main()
├── config.py            # load_settings(): env → Settings (frozen dataclass)
├── client.py            # KanboardClient.call(method, params) → result | raise
├── errors.py            # KanboardError, AuthError, NotFoundError, ConfirmRequiredError
├── guards.py            # require_confirm(confirm, action)
├── server.py            # FastMCP("kanboard"), eager `mcp` for `mcp dev`
├── smoke.py             # mcp-kanboard-smoke entry — getVersion + getAllProjects
└── tools/               # one module per Kanboard domain, each exports register(mcp, client)
```

## Auth model

`KanboardClient` uses HTTP Basic. Two token types both work transparently:

| Token | Username | Source |
|---|---|---|
| Application | `jsonrpc` (literal) | Kanboard → Settings → API |
| Personal | user's login | Kanboard → My Profile → Actions → API |

Env vars (all read by `config.load_settings()`):
- `KANBOARD_URL` (required) — `/jsonrpc.php` suffix appended if missing
- `KANBOARD_API_TOKEN` (required)
- `KANBOARD_USERNAME` (default `jsonrpc`)
- `KANBOARD_TIMEOUT` (default 30)
- `KANBOARD_MAX_ATTACHMENT_MB` (default 25 — applies to both upload + download)
- `KANBOARD_VERIFY_SSL` (default true; set false for self-signed remote)
- `MCP_PASSPHRASE` (required in `--http` mode unless `--insecure-no-auth`) — gates the OAuth login form claude.ai redirects users to
- `MCP_HTTP_HOST` / `MCP_HTTP_PORT` (default `127.0.0.1` / `8765`) — bind for `--http` mode

Env vars consumed by `scripts/start-web.ps1` only (not the server):
- `MCP_PORT` — port override when run without `-Port`
- `KANBOARD_NGROK_AUTHTOKEN` — passed to ngrok as `--authtoken`. Use when another MCP on this machine also uses ngrok and needs a different account. Falls back to `NGROK_AUTHTOKEN`, then ngrok global config.
- `KANBOARD_NGROK_DOMAIN` — static domain (no `https://` prefix). Passed as `--url https://<domain>`. **Required** when the authtoken's account has another agent running, otherwise ngrok picks the wrong default domain → `ERR_NGROK_334`.

## Tool conventions

- **Names**: `kb_<verb>_<noun>` snake_case. `kb_` prefix is the MCP-level disambiguator.
- **Signatures**: positional Kanboard required params first; optional params as keyword args defaulting to `None`. Only non-None values are forwarded — Kanboard doesn't tolerate stray nulls in many endpoints.
- **Returns**: domain-shaped dicts/lists/ints (e.g. `kb_create_task` returns the new id). Destructive tools return `{"removed": bool, "<id_field>": int}`.
- **Errors**: bubble `KanboardError` subclasses; the server-level FastMCP wrapper converts them to clean tool errors.

## Adding a new tool

The pattern is mechanical. See `src/mcp_kanboard/tools/comments.py` for a small reference. To add e.g. `kb_get_task_links`:

1. Pick the right module under `tools/` (or create one and register it in `tools/__init__.py`).
2. Add a function decorated with `@mcp.tool()`. Type hints + the docstring drive the schema and the description the model sees.
3. Translate to Kanboard's JSON-RPC method via `client.call("methodName", {param: value, ...})`.
4. If destructive: take `confirm: bool = False`, call `require_confirm(confirm, "<verb> <subject>")` first, and return `{"removed": ok, "<id>": id_value}`.
5. Smoke-test via `uv run mcp dev src/mcp_kanboard/server.py` (MCP Inspector) before pushing.

## Local development

```powershell
uv sync                                       # install deps into .venv
Copy-Item .env.example .env                   # fill in KANBOARD_URL + KANBOARD_API_TOKEN
uv run mcp-kanboard-smoke                     # auth + connectivity check
uv run mcp dev src\mcp_kanboard\server.py     # MCP Inspector in browser
```

The smoke test calls `getVersion` + `getAllProjects` and exits non-zero on failure — the fastest way to catch auth/URL mistakes.

## Release flow

Distribution is `uvx` from the GitHub repo — no PyPI publish needed.

1. Edit code in `src/mcp_kanboard/`.
2. Commit + push to `main` on `github.com/mrquanga3/mcp-kanboard`.
3. Users (or you) refresh the cached install:
   ```powershell
   uvx --refresh --from git+https://github.com/mrquanga3/mcp-kanboard mcp-kanboard --help
   ```
   Then restart Claude Code. The MCP host will respawn the server from the new commit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrquanga3/mcp-kanboard](https://github.com/mrquanga3/mcp-kanboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
