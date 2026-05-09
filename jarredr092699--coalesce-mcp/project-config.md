---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

An MCP (Model Context Protocol) server that gives Claude read/write access to [Coalesce](https://coalesce.io) — a Snowflake-native data transformation platform. The primary use case is **failure investigation**: ask Claude to diagnose why a pipeline run failed without leaving the chat.

**Package name:** `coalesce-mcp` | **PyPI:** `pip install coalesce-mcp` | **Version:** 0.3.0

---

## Development Commands

```bash
# Install dev dependencies
uv sync

# Run locally
COALESCE_API_TOKEN=xxx uv run coalesce-mcp-server

# Build
uv build

# Publish
uv publish
```

Python 3.10+ required. Key deps: `mcp>=1.0.0`, `httpx>=0.27.0`.

---

## Architecture

**Two files that matter:**
- `server.py` — declares MCP tools (names, descriptions, JSON schemas) and routes `call_tool` invocations to client functions
- `client.py` — `CoalesceClient` class wraps `httpx.AsyncClient`; standalone async functions are the actual MCP tool implementations

**Two-layer tool pattern:**
1. `CoalesceClient.method()` — raw HTTP call, returns `dict`, lets exceptions propagate
2. `tool_function()` — MCP-facing, calls client, formats/filters, returns `str` (JSON); catches `httpx.HTTPStatusError` and returns JSON error objects

**Client singleton:** `get_client()` in `client.py` returns a module-level `CoalesceClient`. The `httpx.AsyncClient` inside is lazily initialized and reused.

**Server routing:** `call_tool(name, arguments)` → appropriate MCP tool function → `[TextContent(type="text", text=result)]`

---

## Configuration

| Variable | Default | Notes |
|---|---|---|
| `COALESCE_API_TOKEN` | (required) | Bearer token — set in `.env` or directly in the MCP host config |
| `COALESCE_BASE_URL` | `https://app.coalescesoftware.io/api` | Override for on-prem |
| `COALESCE_READONLY_MODE` | `false` | Set `true` to hide `create_workspace_node`, `set_node`, `patch_node_field`, `start_run`, `retry_run`, and `cancel_run` tools |

`COALESCE_READONLY_MODE` is used in the Snowflake Cortex CLI integration — the agent has a `DATAENG_READ_ONLY` Snowflake role and the readonly mode prevents write tool exposure.

**Switching accounts:** Update `COALESCE_API_TOKEN` in `.env` (for local dev) or in `~/.snowflake/cortex/mcp.json` (for Cortex CLI), then restart the MCP host.

**Note:** The Cortex CLI does **not** expand `${VAR}` references in `mcp.json` — paste token values directly.

Claude Desktop config snippet:
```json
{
  "mcpServers": {
    "coalesce": {
      "command": "uvx",
      "args": ["--from", "coalesce-mcp", "coalesce-mcp-server"],
      "env": {
        "COALESCE_API_TOKEN": "your-token-here"
      }
    }
  }
}
```

---

## API Endpoints

All calls go to `COALESCE_BASE_URL`. Auth is `Authorization: Bearer <token>`.

### Job Runs (read-only)
| Method | Path | Purpose |
|---|---|---|
| GET | `/v1/runs` | List runs; params: `environmentID`, `runStatus`, `limit`, `startingFrom`, `orderBy` |
| GET | `/v1/runs/{runID}` | Single run details |
| GET | `/scheduler/runStatus?runID={id}` | Live status (different base path than run details) |
| GET | `/v1/runs/{runID}/results` | Node-level execution results — flat dict keyed by node ID |

### Job Run Execution (write)
| Method | Path | Purpose |
|---|---|---|
| POST | `/scheduler/startRun` | Start a new run; body: `environmentID` (required), `jobID`, `parallelism` (optional) |
| POST | `/scheduler/rerun` | Retry a failed run from failure point; body: `runID` |
| POST | `/scheduler/cancelRun` | Cancel an in-progress run; body: `runID`; returns 204 No Content |

### Node Management (read + write)
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/environments/{envID}/nodes` | All deployed nodes |
| GET | `/api/v1/workspaces/{wsID}/nodes` | All workspace nodes |
| GET/PUT | `/api/v1/workspaces/{wsID}/nodes/{nodeID}` | Get or full-replace workspace node |
| GET | `/api/v1/environments/{envID}/nodes/{nodeID}` | Single environment node |
| POST | `/api/v1/workspaces/{wsID}/nodes` | Create node |

---

## MCP Tools Exposed

### Failure Investigation
| Tool | Purpose |
|---|---|
| `list_job_runs` | List runs with optional filters |
| `list_failed_runs` | Shortcut: failed runs only |
| `start_run` | Start a fresh job run (environment or specific job) |
| `retry_run` | Re-run only failed nodes from a prior run — use after patching to verify fix |
| `cancel_run` | Abort an in-progress run |
| `get_run` | Full run object |
| `get_run_status` | Live status via scheduler endpoint |
| `get_run_results` | Pre-processed: failed nodes + blocked downstream + summary stats |
| `get_job_details` | Combined: run info + status + full results + extracted errors |
| `investigate_failure` | **Best for diagnosis:** run metadata + failures + downstream impact |

### Node Management
| Tool | Purpose |
|---|---|
| `list_environment_nodes` / `list_workspace_nodes` | List all nodes |
| `get_workspace_node` / `get_environment_node` | Full node config + SQL |
| `create_workspace_node` | Create with defaults |
| `set_node` | Full replacement update (read current first!) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JarredR092699/coalesce-mcp](https://github.com/JarredR092699/coalesce-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
