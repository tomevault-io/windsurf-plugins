---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

apex-mcp is an MCP (Model Context Protocol) server for Oracle APEX 24.2 that exposes 116+ tools to AI clients (Claude, GPT, Gemini, Cursor, VS Code). It generates APEX applications programmatically via PL/SQL using the `wwv_flow_imp_page.*` internal API (same as APEX's own import/export).

## Build & Run

```bash
# Install
pip install -e ".[dev]"

# Run (stdio — default for local AI clients)
python -m apex_mcp
apex-mcp                    # equivalent, via entry point

# Run with HTTP transport
python -m apex_mcp --transport streamable-http --port 8000
python -m apex_mcp --transport sse --port 9000
```

## Testing

```bash
# Unit tests only (no Oracle DB required)
pytest tests/ -v -m "not integration"

# All tests (requires live Oracle ADB + env vars)
pytest tests/ -v

# Single test file
pytest tests/test_session.py -v

# Single test
pytest tests/test_ids.py::test_unique_ids -v
```

Tests require `ORACLE_DB_USER`, `ORACLE_DB_PASS`, `ORACLE_DSN`, `ORACLE_WALLET_DIR`, `ORACLE_WALLET_PASSWORD`, `APEX_WORKSPACE_ID`, `APEX_SCHEMA`, `APEX_WORKSPACE_NAME` env vars for integration tests.

## Architecture

### Core Singletons (module-level instances)

- **`db`** (`apex_mcp/db.py`) -- `ConnectionManager` singleton. Thread-safe Oracle connection via oracledb thin driver + mTLS wallet. Has auto-reconnect on transient ORA errors (03113, 03114, 12170, 25408), dry-run mode (logs PL/SQL without executing), and batch mode (queues PL/SQL, executes in one commit). Also provides `safe_col()` / `column_exists()` for APEX version-safe column access with caching.
- **`session`** (`apex_mcp/session.py`) -- `ImportSession` singleton. Tracks all components (pages, regions, items, buttons, LOVs, auth schemes, charts, processes, branches, dynamic actions) created during the `apex_create_app()` to `apex_finalize_app()` lifecycle. Tools use it to resolve cross-references like region IDs. Thread-safe via `threading.RLock`.
- **`ids`** (`apex_mcp/ids.py`) -- `IdGenerator` singleton. Produces unique IDs (base 8,900,000,000,000,000 + random salt + counter) with a named registry to avoid collisions within a session. Reset on each `apex_create_app()` call.

### Supporting Modules

- **`config.py`** -- Reads all env vars at import time; emits `RuntimeWarning` for missing vars (does not crash, so unit tests work without Oracle).
- **`templates.py`** -- Hardcoded Universal Theme 42 template IDs (page, region, button, label, list, report templates) with `discover_template_ids()` to refresh from the live database.
- **`themes.py`** -- Complete Unimed-branded CSS theme (palette `#00995D`) for Universal Theme 42 / Redwood Light.
- **`validators.py`** -- Input validation for page_id, app_id, region_name, sql_query, chart_type, item_type, table_name, color_hex, sequence. All raise `ValueError` with descriptive messages.
- **`guards.py`** -- Pre-condition checks (`require_connection`, `require_session`, `require_page`) returning JSON error strings or `None`.
- **`exceptions.py`** -- Domain exceptions: `ApexMCPError`, `NotConnectedError`, `NoSessionError`, `PageNotFoundError`, `RegionNotFoundError`.
- **`utils.py`** -- `_esc()` (PL/SQL quote escaping), `_blk()` (anonymous block wrapper), `_json()` (JSON serialization with `ensure_ascii=False`), `_sql_to_varchar2()` (converts multi-line SQL to `wwv_flow_string.join(...)` expressions).

### App Creation Lifecycle

Every app build follows: `apex_connect()` -> `apex_create_app()` -> [add pages/regions/items/etc.] -> `apex_finalize_app()`. The session singleton tracks state between these calls. `finalize` closes the import and commits.

### Tool Modules (`apex_mcp/tools/`)

17 modules (+ `__init__.py`), each containing related MCP tool functions. All tools are plain async/sync functions registered in `server.py` via `mcp.tool()`. They share access to `db` and `session` singletons.

### PL/SQL Generation Pattern

All tools that create APEX components follow the same pattern:
1. Generate a unique ID via `ids`
2. Build PL/SQL calling `wwv_flow_imp_page.*` or `wwv_flow_imp_shared.*` procedures
3. Execute via `db.plsql()` (respects dry-run/batch modes)
4. Register the component in `session` for cross-reference tracking

### Server Registration (`apex_mcp/server.py`)

The FastMCP server imports all tool functions and registers them with `mcp.tool()`. Each registration uses `description=` to provide a concise one-liner (~65 chars avg) that overrides the verbose Python docstring in the MCP tool listing sent to AI clients. This reduces LLM context cost from ~84K chars to ~10K chars (89% reduction). The `instructions` string serves as the system prompt sent to AI clients — kept compact (~1.7K chars) with lifecycle, quick-build pattern, generators reference, and conventions.

## MCP Configuration (`.mcp.json`)

The project root contains `.mcp.json` for IDE integration (Cursor, VS Code, etc.):

```json
{
  "mcpServers": {
    "apex-mcp": {
      "command": "python",
      "args": ["-m", "apex_mcp"],
      "cwd": "${workspaceFolder}",
      "env": {
        "ORACLE_DB_USER": "YOUR_SCHEMA",
        "ORACLE_DB_PASS": "YOUR_PASSWORD",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TechFernandesLTDA/apex-mcp](https://github.com/TechFernandesLTDA/apex-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
