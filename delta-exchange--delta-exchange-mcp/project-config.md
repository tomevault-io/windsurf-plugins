---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project in one line

FastMCP server (stdio only) that wraps Delta Exchange India's REST API as MCP tools — public market data unconditionally, authenticated read-only account tools when `DELTA_API_KEY`/`DELTA_API_SECRET` are set, plus authenticated trading mutations when `DELTA_MCP_MODE=trade` is also set.

## Style

Don't add typing slop. In particular:

- Don't annotate pytest fixtures (`tmp_path`, `monkeypatch`, etc.) — pytest discovers them by name, the annotation adds nothing.
- Don't write `**kwargs: Any` / `-> Any` on internal test helpers. If the only honest type is `Any`, leave it off.
- Use `Any` only when it carries real information: a public boundary that genuinely accepts arbitrary JSON, a return type that is genuinely heterogeneous. Otherwise prefer the real type or no annotation at all.
- Don't add `from typing import Any` just to satisfy a redundant annotation.

## Commands

```bash
uv sync                                        # install deps (runtime + dev)
uv run pytest                                  # run full suite (asyncio_mode=auto)
uv run pytest tests/test_market_tools.py::test_429_retries_then_succeeds  # single test
uv run ruff check src tests scripts            # lint
uv run ruff check --fix src tests scripts      # lint + autofix

uv run delta-exchange-mcp                      # stdio (the only transport)

uv run python scripts/smoke.py                 # live smoke against DELTA_MCP_ENV

bash scripts/inspect.sh --cli --method tools/list
bash scripts/inspect.sh --cli --method tools/call --tool-name get_ticker --tool-arg symbol=BTCUSD
bash scripts/inspect.sh                                                          # Inspector web UI on :6274
```

**Rebuilding the editable install after changing `pyproject.toml` or entry points**: `uv sync` again — `uv run` caches the build.

## Architecture

### Tool registration pattern

Each tool module exposes `register(mcp: FastMCP, client: DeltaClient) -> None` that attaches `@mcp.tool()`-decorated closures. `server.py::build_server()` instantiates `DeltaClient` once and passes it into every `register` call. **To add a tool group**: create `src/delta_exchange_mcp/tools/<group>.py` with a `register(mcp, client)`, then call it from `build_server`.

`market.register` always runs; `account.register` starts only when `cfg.has_credentials` is true (both `DELTA_API_KEY` **and** `DELTA_API_SECRET` set), and reconciliation can add or remove that whole manifest later.

### Bringing the account surface up without a restart

A credential saved through the in-chat form arrives in the running process, so `build_server` closes over an `activate(session)` callback and hands it to `form.register`. Every tool closure holds the same rebindable `DeltaClient`: reconciliation swaps one immutable `{config, signing path, http client}` state so market and account calls move to a new environment or credential pair together. It then adds or removes the complete account-tool manifest, disarms trading before any identity change, and sends `session.send_tool_list_changed()` whenever the surface changes. `activate` returns the account and trading surfaces that are actually live, which is what drives the form's carry-on/restart copy.

Three things here are load-bearing:

- **The capability has to be declared.** `serve()` runs stdio with `initialization_options(mcp)`, which passes `NotificationOptions(tools_changed=True)`. FastMCP's own `run_stdio_async` leaves every flag off, so the server would advertise `tools.listChanged: false` and a client would never re-read the tool list — the notification would be silently useless. `main` therefore calls `anyio.run(serve, mcp)`, **not** `mcp.run()`. Regression test: `test_the_server_declares_that_its_tool_list_can_change`.
- **Trade mode is never armed by a form save.** `activate` hot-applies reads and the safe trade→read direction, but read→trade waits for a new session's first `tools/list`. Regression tests: `test_trade_mode_still_waits_for_a_restart`, `test_mode_only_read_disarms_live_trading_immediately`.
- **Rotation and environment changes are coherent hot changes.** The shared client swaps its entire request identity before the next call; an in-flight request keeps the state it captured. If trading was live, its tools are removed before that swap and require a restart to re-arm. Regression tests: `test_a_rotated_key_signs_the_next_account_request`, `test_the_first_save_rebinds_market_and_account_tools_to_one_environment`, `test_external_identity_drift_disarms_trading_before_hot_rebind`.
- **Runtime transitions are observable without secrets.** Startup and each trade arm/disarm write one structured line to stderr with environment, live mode, registered surface, and audit path. Never add keys, secrets, signatures, or credential fingerprints to this output.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [delta-exchange/delta-exchange-mcp](https://github.com/delta-exchange/delta-exchange-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
