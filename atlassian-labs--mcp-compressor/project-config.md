---
trigger: always_on
description: This file provides guidance for coding agents working in this repository.
---

# AGENTS.md

This file provides guidance for coding agents working in this repository.

## Repo purpose

`mcp-compressor` is a Python CLI and MCP proxy server that wraps an upstream MCP server and reduces the token footprint exposed to LLMs.

At a high level it:
- connects to an upstream MCP server over stdio, streamable HTTP, or SSE
- proxies that server through FastMCP
- replaces a large tool surface with a compressed wrapper interface
- optionally supports CLI mode and TOON output formatting
- persists OAuth state for remote servers using encrypted local storage

## Important source files

- `mcp_compressor/main.py`
  - CLI entrypoint (`main`, `entrypoint`)
  - transport selection and creation
  - proxy server startup (`_server`, `_cli_mode_server`, `_proxy_client`)
  - OAuth storage helpers (`_build_token_storage`, `_get_or_create_encryption_key`)
  - `clear-oauth` subcommand
- `mcp_compressor/tools.py`
  - `CompressedTools` middleware class
  - compressed tool listing/schema lookup/invocation
  - include/exclude tool filtering
  - validation error formatting
  - TOON output conversion
- `mcp_compressor/logging.py`
  - `configure_logging(log_level)` — loguru setup and stdlib logging intercept
  - `suppress_recoverable_oauth_traceback_logging(transport)` — narrow log filter context manager for recoverable OAuth retries
  - `_RecoverableOAuthTracebackFilter` — the log filter implementation
- `mcp_compressor/cli_tools.py`
  - CLI-facing tool help and argument handling
- `mcp_compressor/cli_bridge.py`
  - local HTTP bridge used in CLI mode
- `mcp_compressor/cli_script.py`
  - generated CLI script management
- `mcp_compressor/types.py`
  - enums and shared types (`CompressionLevel`, `LogLevel`, `TransportType`)
- `tests/`
  - unit/integration coverage for transports, middleware, CLI mode, and proxy behavior

## How the proxy works

### Connection setup

`main()` → `_async_main()` → `_server()` or `_cli_mode_server()`

1. Infers transport type from the command/URL argument
2. Creates a FastMCP transport (stdio, streamable HTTP, or SSE)
3. Opens a `ProxyClient` via `_proxy_client(transport)`, which:
   - wraps the connection with log suppression for recoverable OAuth errors
   - on failure matching the stale-OAuth-500 signature, clears cached OAuth state and retries once
4. Creates a proxy `FastMCP` server via `FastMCP.as_proxy(backend=client)`
5. Installs `CompressedTools` middleware on the proxy server

### Compression middleware

`CompressedTools` (in `tools.py`) is the core behavior of this repo.

It:
- hides all upstream backend tools from clients
- registers a small fixed set of wrapper tools instead:
  - `get_tool_schema(tool_name)` — returns the full upstream schema on demand
  - `invoke_tool(tool_name, tool_input)` — calls the upstream tool
  - `list_tools()` — returned only at `max` compression level
- formats compressed tool listings at four levels: `low`, `medium`, `high`, `max`
- applies optional include/exclude filters on the backend tool set
- enriches validation failures with the real upstream schema
- optionally converts JSON text outputs to TOON format
- exposes a hidden resource `compressor://uncompressed-tools` with the raw upstream tool list

### CLI mode

CLI mode exposes a single `<server>_help` tool to the LLM, and generates a local shell script that maps subcommands to upstream tool invocations via a local HTTP bridge.

Key files:
- `mcp_compressor/cli_tools.py` — help text and arg parsing
- `mcp_compressor/cli_bridge.py` — local HTTP bridge
- `mcp_compressor/cli_script.py` — shell script generation and lifecycle management

### OAuth and token persistence

For remote (HTTP/SSE) servers, `mcp-compressor`:
- delegates the OAuth flow entirely to FastMCP's `OAuth` client
- adds encrypted persistent token storage via:
  - OS keyring (preferred) or file-based fallback for the encryption key
  - `py-key-value-aio` + `cryptography.fernet` for the encrypted token store
- provides a `clear-oauth` CLI command to reset cached state
- suppresses noisy upstream traceback logs when a stale-cache OAuth error is automatically recovered

## Core architectural patterns

### 1. Prefer thin integration code over protocol reimplementation
This project relies heavily on FastMCP and the MCP Python SDK. Prefer using their built-in types and flows rather than reimplementing protocol behavior locally.

Examples:
- use FastMCP transports and `ProxyClient`
- use FastMCP `OAuth` for the OAuth flow; local code only adds persistent storage and clear-oauth UX
- keep local logic focused on compressed tool exposure, encrypted token persistence, and CLI UX

### 2. Keep changes narrow and composable
The repo is relatively small and organized around a few key flows. Prefer adding small helpers over broad refactors unless a broader change is clearly justified.

Good examples already in the codebase:
- small transport helper functions in `main.py`
- encapsulated middleware logic in `CompressedTools`
- targeted helpers for OAuth cache clearing and retry behavior in `main.py`
- logging setup and suppression isolated in `logging.py`

### 3. Preserve pass-through semantics
This wrapper should generally preserve upstream behavior unless it is intentionally transforming output.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atlassian-labs/mcp-compressor](https://github.com/atlassian-labs/mcp-compressor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
