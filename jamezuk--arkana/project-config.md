---
trigger: always_on
description: Arkana is a Model Context Protocol (MCP) server exposing 294 binary analysis tools to AI clients. It supports PE, ELF, and Mach-O formats with integrations for angr, capa, FLOSS, YARA, Binary Refinery, Qiling, Speakeasy, oletools, and GoReSym.
---

# Arkana Development Guide

## What is Arkana?

Arkana is a Model Context Protocol (MCP) server exposing 294 binary analysis tools to AI clients. It supports PE, ELF, and Mach-O formats with integrations for angr, capa, FLOSS, YARA, Binary Refinery, Qiling, Speakeasy, oletools, and GoReSym.

## Project Structure

```
arkana/                  # Main package
├── main.py             # Entry point, arg parsing, server startup
├── state.py            # Thread-safe AnalyzerState + StateProxy (per-session isolation)
├── config.py           # Central re-export hub (constants, imports, state, cache)
├── constants.py        # Pure constants (response limits, timeouts, URLs)
├── imports.py          # Optional library imports with *_AVAILABLE flags
├── cache.py            # Gzip-compressed LRU disk cache (~/.arkana/cache/)
├── auth.py             # Bearer token ASGI middleware
├── integrity.py        # Pre-parse file integrity checks (PE/ELF/Mach-O)
├── utils.py            # ReDoS-safe regex, safe_slice, safe_env_int
├── tool_registry.py    # Tool module groups and startup registration
├── parsers/            # PE/FLOSS/capa/YARA/strings parsers
├── dashboard/          # Web dashboard (Starlette + htmx + Jinja2)
│   ├── app.py          # ASGI app factory, routes, auth, SSE events
│   ├── state_api.py    # Data extraction layer (reads AnalyzerState for dashboard views)
│   ├── __init__.py     # Package init
│   ├── templates/      # Jinja2 templates (overview, functions, callgraph, sections, strings, timeline, notes)
│   │   └── partials/   # htmx partials (_global_status, _overview_stats, _task_list, _timeline_entry)
│   └── static/         # CSS (CRT theme), JS (htmx, Cytoscape.js, strings.js), logo
├── resource_monitor.py  # Process-level RSS/CPU monitoring (psutil daemon thread)
    └── mcp/                # MCP tool modules (294 tools across 65 files)
    ├── server.py       # FastMCP instance, tool_decorator, response truncation
    ├── _*.py           # Private helpers (angr, input, format, progress, refinery, rename, search)
    └── tools_*.py      # Tool modules grouped by domain
tests/                  # Unit tests (pytest)
tests/integration/      # Integration tests (requires running server)
.claude/skills/         # Claude Code analysis and learning skills
```

## Running Tests

```bash
# Unit tests (no server needed, ~2 seconds)
python -m pytest tests/ -v

# Unit tests with coverage
python -m pytest tests/ -v --cov=arkana --cov-config=.coveragerc

# Integration tests (requires running server)
./run.sh  # Start server in one terminal
python -m pytest tests/integration/mcp_test_client.py -v  # In another
```

Coverage configuration lives in `.coveragerc` (single source of truth). MCP tool modules are excluded from unit test coverage — they are tested via integration tests.

## Lint

```bash
ruff check arkana/ tests/ \
  --select=E9,F63,F7,F82,F841,W291,W292,W293,B006,B007,B018,UP031,UP032,RUF005,RUF010,RUF019,G010
```

## Key Patterns

- **File integrity checks**: `arkana/integrity.py` validates binaries pre-parse using only `struct`. `open_file` runs this automatically. The `force` parameter overrides smart fallback. `open_file`/`close_file` **block when background tasks are active** — they return an error with `active_tasks` and `hint` suggesting `abort_background_task()` or `force_switch=True`. When `open_file` is called on a new file without `close_file()`, it clears all module-level caches to prevent cross-file data contamination. `open_file` PE analysis uses soft/overtime timeouts (task ID `"pe-analysis"`): `PE_ANALYSIS_SOFT_TIMEOUT` (300s) → `TASK_OVERTIME` with stall detection → `PE_ANALYSIS_MAX_RUNTIME` (3600s) ceiling. Set `ARKANA_PE_ANALYSIS_SOFT_TIMEOUT=0` for old hard-timeout behavior.
- **Optional deps**: Every library is guarded by `*_AVAILABLE` flags in `imports.py`. Tools return actionable error messages when a dep is missing — never crash.
- **PE object guard**: `_check_pe_object(tool_name, require_headers=False)` in `server.py` validates `state.pe_object` is not `None` before tools access it. Use `require_headers=True` for tools needing `OPTIONAL_HEADER`/`FILE_HEADER` (rejects ELF/Mach-O/shellcode). Call after `_check_pe_loaded()`.
- **Thread safety**: All shared state uses locks. `StateProxy` + `contextvars` isolates HTTP sessions. `_cached_*` fields on `AnalyzerState` rely on CPython's GIL for atomic reference replacement (never mutated in place after assignment).
- **Session limits**: `MAX_ACTIVE_SESSIONS` (default 100, env: `ARKANA_MAX_SESSIONS`). Per-session caps: `MAX_NOTES` (10K), `MAX_TOOL_HISTORY` (500), `MAX_ARTIFACTS` (1K), `MAX_RENAMES` (10K), `MAX_COMPLETED_TASKS` (50).
- **Response truncation**: Dual-limit — soft char limit (8K default) plus hard byte limit (64KB). `tool_decorator` auto-enforces even for tools that don't call `_check_mcp_response_size`. Set `ARKANA_MCP_RESPONSE_LIMIT_CHARS=65536` for non-Claude-Code clients.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JameZUK) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
