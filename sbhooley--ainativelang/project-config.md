---
trigger: always_on
description: > Read this FIRST before doing anything in this repository.
---

# AGENTS.md — Ground Truth for AI Agents

> Read this FIRST before doing anything in this repository.

## What This Repo Is

Python compiler + runtime for AINL (AI Native Language), version 1.7.0.
AINL compiles `.ainl` source files into an IR (intermediate representation)
graph, then executes that graph via adapters (database, HTTP, LLM, Solana, etc).

## Repository Layout

```
compiler_v2.py          — The compiler (5300 lines). Parses .ainl → IR dict.
compiler_diagnostics.py — Error/warning types used by compiler.
runtime/engine.py       — The runtime engine (2150 lines). Executes IR graphs.
runtime/adapters/       — Runtime adapter base classes and builtins.
cli/main.py             — CLI entry point (2700 lines). All `ainl` commands.
adapters/               — 54 adapter modules (solana, postgres, redis, LLM, etc); ArmaraOS monitor bootstrap: `armaraos_integration.py`, `armaraos_defaults.py` (`build_armaraos_monitor_registry`, `boot_armaraos_graph_memory`). See `docs/ARMARAOS_INTEGRATION.md`, `docs/adapters/AINL_GRAPH_MEMORY.md`.
armaraos/emitter/       — `armaraos.py`: `ainl emit --target armaraos` Hand pack (`HAND.toml`, IR JSON, `security.json`, README) with `schema_version` for openfang-hands validation.
scripts/                — Standalone scripts (emit_langgraph, emit_temporal, etc).
tooling/                — Graph analysis, normalization, effect analysis tools.
examples/               — 154+ .ainl example files. **Training contract:** only paths listed as `strict-valid` in `tooling/artifact_profiles.json` are CI-checked with `ainl validate --strict`. See `examples/README.md` and `docs/EXAMPLE_SUPPORT_MATRIX.md` before copying random files.
tests/                  — ~1000 test files, 300K lines. Run with pytest.
docs/                   — Documentation (some accurate, some aspirational — see below).
```

## Commands That Work

```bash
ainl run <file>                          # Compile and execute
ainl validate <file> [--strict]          # Validate (alias for check)
ainl validate <file> --json-output       # Full IR JSON output
ainl compile <file>                      # Compile to IR JSON
ainl emit <file> --target <t> [-o path]  # Emit to target platform
ainl serve [--host H] [--port P]         # HTTP server (REST API)
ainl check <file> [--strict]             # Same as validate
ainl visualize <file> --output -         # Mermaid diagram
ainl inspect <file>                      # Canonical IR dump
ainl init <name>                         # Create new project
ainl doctor                              # Environment diagnostics
ainl-validate <file> --emit <target>     # Separate script, more emit targets
```

### Emit Targets (ainl emit --target)

```
ir, langgraph, temporal, hermes-skill, hermes,
solana-client, blockchain-client, server, python-api,
react, openapi, prisma, sql, docker, k8s, cron, armaraos
```

**`armaraos`:** emits an **openfang-hands** Hand directory (`HAND.toml`, `<stem>.ainl.json`, `security.json`, `README.md`) with **`schema_version`** on **`[hand]`**, the IR JSON, and **`security.json`** — see **`docs/ARMARAOS_INTEGRATION.md`** § *Emit a Hand package*.

### ainl serve Endpoints

```
GET  /health     — Health check
POST /validate   — Validate source (JSON: {source, strict?})
POST /compile    — Compile to IR (JSON: {source, strict?})
POST /run        — Compile and execute (JSON: {source, strict?, frame?})
```

### Production defaults (runner + mass-market UX)

- **Default runner grant** (`scripts/runtime_runner_service.py`): adapter cap is **unset** at the grant layer so IR-declared adapters (e.g. `web`, `http`) work out of the box; **resource limits** (`max_steps`, `max_time_ms`, …) remain the universal safety floor.
- **MCP server** (`scripts/ainl_mcp_server.py`): **`ainl_run`** only registers `core` by default. Any workflow using `http`, `fs`, `cache`, or `sqlite` **MUST** pass them via the `adapters` parameter — otherwise the run fails with "adapter not registered". Example for a workflow that does HTTP + file I/O + caching:
  ```json
  {
    "enable": ["http", "fs", "cache"],
    "http": { "allow_hosts": ["example.com", "api.otherdomain.gov"], "timeout_s": 15 },
    "fs":   { "root": "/absolute/path/to/workspace", "allow_extensions": [".json", ".csv"] },
    "cache":{ "path": "/absolute/path/to/workspace/cache.json" }
  }
  ```
  LLM adapters additionally require **`AINL_CONFIG`** or **`AINL_MCP_LLM_ENABLED=1`** (see `docs/LLM_ADAPTER_USAGE.md`).
- **MCP authoring loop:** `ainl_validate` / `ainl_compile` use **`CompilerContext`** so diagnostics include native **`kind`**, **`suggested_fix`**, and related fields. Responses include **`recommended_next_tools`** (and sometimes **`recommended_resources`**); fetch MCP resource **`ainl://authoring-cheatsheet`** for HTTP `R`-line and adapter rules. **`ainl_capabilities`** also returns **`mcp_telemetry`** (per-process call counters: validate/compile/run and “run after validate without compile”).
- **`AINL_STRICT_MODE=1`** (when **`AINL_SECURITY_PROFILE`** is unset): merges profile **`consumer_secure_default`** (or **`AINL_STRICT_PROFILE`**) with that floor — named allowlist + `operator_sensitive` tier blocked; good for “strict” product mode without hand-maintaining env lists.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbhooley/ainativelang](https://github.com/sbhooley/ainativelang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
