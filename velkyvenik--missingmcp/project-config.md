---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A multi-user, OAuth 2.1–protected **remote MCP gateway** that lets a small trusted circle each connect their own upstream-service accounts to Claude (mobile/desktop/web). The gateway terminates OAuth, performs the adapter-specific login — a credential form (garmin) or a redirect to the upstream's own OAuth (whoop: `/whoop/oauth/callback`) — stores per-account encrypted blobs, and forwards `/<adapter>/mcp` via one of three strategies: **worker** (garmin — spawns + reverse-proxies to a per-user subprocess of the **unmodified** `garmin_mcp` worker, `github.com/Taxuspt/garmin_mcp`), **remote** (no subprocess; forwards to a hosted upstream MCP, injecting the account's credentials as headers), or **local** (whoop — no subprocess, no shared upstream; the MCP server runs in-process, see `adapters/whoop/mcp.py`). No in-tree adapter uses the remote strategy today — rohlik used it until Rohlík shipped its own OAuth MCP (2026-07); the strategy stays covered by `tests/test_remote_forward.py` via a stub adapter.

The canonical design and the task-by-task implementation plan live in `docs/superpowers/specs/` and `docs/superpowers/plans/` — read them for rationale and the full data flow, but treat them as dated design records: the 2026-07-05 multi-adapter spec still describes a rohlik adapter that was implemented and then retired (2026-07-06, Rohlík ships its own OAuth MCP) — don't re-add it. Operator-facing docs (env-var reference, monitoring, deploy checklist) live in `README.md`; operational scripts (`status`, `revoke`, `usage`) live in `scripts/` and are documented in README → Monitoring.

## Commands

```bash
# Tests — the `--extra dev` is REQUIRED: pytest lives in [project.optional-dependencies].dev,
# so plain `uv run pytest` fails with "no module named pytest".
uv run --extra dev pytest -q                          # full suite
uv run --extra dev pytest tests/test_oauth.py -v      # one file
uv run --extra dev pytest tests/test_oauth.py::test_metadata_shape -v   # one test

# Run the gateway locally (no Garmin needed to exercise the OAuth surface).
# DATA_DIR defaults to /data (not writable locally) — point it somewhere writable.
# GATEWAY_SECRET must be >=32 chars AND must not start with "change-me" (startup guard).
# To exercise the full /<adapter>/mcp path locally, also set GARMIN_MCP_CMD (garmin-mcp isn't on
# PATH): GARMIN_MCP_CMD="uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp"
GATEWAY_SECRET="$(openssl rand -base64 48)" PUBLIC_URL=http://localhost:8088 PORT=8088 \
  DATA_DIR=./.localdata uv run missingmcp

# After changing adapters/whoop/mcp.py's TOOLS table, regenerate the landing
# page's tool listing:
python scripts/gen_whoop_tools.py

# After changing the link-preview card's copy or palette, redraw static/og.png
# (Pillow is not a project dependency, hence --with). The `?v=` cache-buster in
# pages.py is derived from the file's hash, so a redraw invalidates scraper caches
# by itself:
uv run --with pillow python scripts/gen_og_image.py

# Production (missingmcp.com) runs on Railway, built from the Dockerfile, and
# auto-deploys on every push to main — pushing = deploying. Verify after push:
# railway deployment list --json. (Self-host: plain `docker run` — see README.)
```

There is no separate lint step configured.

## Hard constraints

- **Never modify or import `garmin_mcp`.** Interact with it *only* as a black box via its documented CLI entrypoint (`garmin-mcp`) and env vars (`GARMIN_MCP_TRANSPORT`, `GARMIN_MCP_HOST`, `GARMIN_MCP_PORT`, `GARMINTOKENS`). No source edits, no importing its internal modules.
- **Pin `GARMIN_MCP_REF`** to a reviewed commit SHA in production (the `main` default is a floating ref — supply-chain risk). After bumping the pin, run `python scripts/gen_garmin_tools.py` — it regenerates the "All tools" section of `src/missingmcp/templates/garmin.html` from the new ref.
- **Python 3.12** (matches the worker's interpreter). All source under `src/missingmcp/`, all tests under `tests/`.

## Architecture

Request flow (one user, one device):

```
Claude → OAuth 2.1 (DCR → /<adapter>/oauth/register → /<adapter>/oauth/authorize → /<adapter>/oauth/token, PKCE S256, RFC 8414 discovery at /.well-known/oauth-authorization-server/<adapter>)
       → adapter-specific login (garmin: garminconnect, password discarded, tokens kept; whoop: redirect to WHOOP's own OAuth, callback at /whoop/oauth/callback)
       → encrypted blob in SQLite, keyed by (adapter, account_key)
       → on POST /<adapter>/mcp, forward strategy (RFC 9728 discovery at /.well-known/oauth-protected-resource/<adapter>/mcp):
           worker (garmin): ensure the user's worker subprocess (127.0.0.1:<port>) → reverse-proxy
           remote (no in-tree adapter today): stream-forward to forward.upstream_url with forward.headers(blob) injected
           local (whoop): forward.handle(conn, account_key, blob, body) runs the MCP server in-process — no subprocess, no upstream_url
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VelkyVenik/missingmcp](https://github.com/VelkyVenik/missingmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
