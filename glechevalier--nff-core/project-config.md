---
trigger: always_on
description: > **Simulation note:** Wokwi simulation was split out of `nff` into the separate **nff-sim**
---

# nff — Rust Architecture

> **Simulation note:** Wokwi simulation was split out of `nff` into the separate **nff-sim**
> package (`../nff-sim`). `nff` is hardware-only — compile, flash, monitor. Nothing about
> Wokwi (the `--sim` flag, `nff wokwi`, diagram.json, the wokwi config/board-chip metadata)
> remains in this repo.

## Status

> **CURRENT (2026-06): the Rust binary (`nff-rs/`) is the shipped product.** `pip install nff`
> now delivers a per-platform wheel containing the compiled Rust binary (maturin `bindings="bin"`,
> see `pyproject.toml`) — no Python runtime. The Rust port is at parity: all CLI commands, both
> build backends (PlatformIO default + arduino), and the full MCP server (HTTP + OAuth proxy +
> `/health` + background-daemon auto-start). The Python package under `nff/` remains as a
> reference/prototype kept in sync version-for-version — **land features in BOTH** so they never
> drift (prototype in Python if you like, but the shipped behavior is Rust).

The Rust port replaces the Python `nff` with a single compiled binary — no Python runtime for end
users, stronger types, better cross-platform packaging.

The MCP server is now native Rust (`nff-rs/nff/src/mcp_server.rs`, rmcp crate). Only
`nff test` still delegates to the Python package via subprocess.

**Adding a new MCP tool (current Python flow):** add an `async def` handler in
`nff/nff/mcp_server.py`, register it in both `_TOOLS` (with an `inputSchema`) and `_DISPATCH`.
Local hardware/toolchain logic lives in `nff/nff/tools/`. *(When the Rust port resumes, the
equivalent is a `#[tool(...)]` method on `NffServer` in `nff-rs/nff/src/mcp_server.rs`.)*

## Claude ↔ nff Handshake

### 1. Registration (`nff init`)

The live Python `nff init` calls `_register_mcp()` (`nff/commands/init.py`) which runs:

```
claude mcp add --scope user --transport http nff http://127.0.0.1:3010/mcp
```

This registers `nff mcp` as a user-scoped, streamable-HTTP MCP server. The transport and
URL are already included; the URL is passed positionally (there is no `--url` flag in this
form). A Bearer `--header` is **not** added here — local bench tools need no auth, and the
diagnosis tools carry their own token when they call the server.

> **Note:** the paused Rust port (`commands/init.rs`, `register_mcp_claude_code()`) was
> specced to register over stdio (`claude mcp add --scope user nff <nff_exe_path> mcp`) and
> still needs to be brought to parity with the Python HTTP form above when that work resumes.

### 2. Transport

`nff mcp` starts a **streamable HTTP MCP server** on `http://127.0.0.1:3010/mcp`
(default; override with `--host` / `--port`). All MCP messages — initialize, tools/list,
tools/call — are HTTP POST requests to that path.

### 3. Bearer authentication (opt-in, OFF by default)

**The `/mcp` Bearer gate is OFF by default.** nff is a single-user, localhost-only bench tool,
so out of the box `/mcp` is open: no token, no OAuth handshake, no "needs authentication". The
server still binds to `127.0.0.1` only, so it is not network-reachable — but any local process
can call the tools.

**Requiring auth (`NFF_MCP_REQUIRE_AUTH`):** set `NFF_MCP_REQUIRE_AUTH=1` (also accepts
`true`/`yes`/`on`) in the environment the server is launched from to turn the gate back ON. When
set, every request to `/mcp` must carry `Authorization: Bearer <token>` validated against the
opaque MCP token (`config.mcp.access_token`) — or, for back-compat, the legacy
`config.diagnosis.access_token` — in `~/.nff/config.json`. A missing or wrong token then returns
HTTP 401 and Claude surfaces an "Unauthorized" error, driving the OAuth browser login. (`/health`
is always unauthenticated, used only for liveness probing.) Gating the tools is the reason the
server is HTTP, not stdio: stdio can't gate them.

Implemented in both `bearer_auth` (Rust, `mcp_server.rs`) and `_NffASGI` (Python, `mcp_server.py`);
the server's advertised `instructions` string reflects whichever mode is active.

**One-time bootstrap order:**

```
1. nff init              # signs you in (browser login, required), detects board,
                         #   writes config, calls _register_mcp()
                         #   (claude mcp add --scope user --transport http nff http://127.0.0.1:3010/mcp),
                         #   then starts the MCP server in the background (daemon.start_background)
2. Restart Claude Code   # Claude picks up the registration and connects to the running server;
                         #   the OAuth proxy fast-path reuses the stored token (no second login)
```

> The background server is started once by `nff init` and runs until reboot. After a reboot,
> `nff mcp` (or re-running `nff init`) brings it back; `nff doctor` reports if it's down.

### 4. Tool call flow

```
Claude Code
    │
    │  HTTP POST  http://127.0.0.1:3010/mcp
    │  Authorization: Bearer <access_token>
    │  Body: MCP tools/call { "name": "...", "arguments": {...} }
    ▼
nff mcp server  (bearer_auth validates token vs ~/.nff/config.json)
    │
    ├──► local tools
    │       list_devices, compile, flash, serial_read, serial_write,
    │       reset_device, get_device_info
    │       — operate on local hardware / toolchain; no further auth
    │
    └──► diagnosis tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GLechevalier/nff-core](https://github.com/GLechevalier/nff-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
