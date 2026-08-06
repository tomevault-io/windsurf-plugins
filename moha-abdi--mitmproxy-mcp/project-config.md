---
trigger: always_on
description: > **Project**: MCP Server for mitmproxy -- HTTP traffic analysis via Model Context Protocol
---

# mitmproxy-mcp Development Guide for AI Agents

> **Project**: MCP Server for mitmproxy -- HTTP traffic analysis via Model Context Protocol
>
> **Status**: Feature-complete (v0.1.0), pre-PyPI publish
>
> **Stack**: Python 3.10+, mitmproxy 11, MCP SDK, Pydantic v2 (v1 compat API)
>
> **Repo**: [moha-abdi/mitmproxy-mcp](https://github.com/moha-abdi/mitmproxy-mcp)

---

## Architecture

MCP server runs inside mitmproxy as an addon. Not a separate process -- it's embedded directly in the proxy event loop via the `running()` hook and `asyncio.create_task()`.

```
mitmproxy (proxy process)
  addon.py (thin wrapper, loads from mitmproxy_mcp.addon)
    MitmproxyAddon
      flow hooks: request(), response(), error()
      MCP server: SSE on port 9011 (default)
        20 tools across 4 categories
```

The SSE transport uses Starlette + uvicorn. Each connecting client gets its own `ServerSession` with a unique session ID -- multi-client is handled natively by the MCP SDK.

---

## Dependency Management

Always use `uv`, never `pip`:

```bash
uv pip install -e ".[dev]"
uv pip install <package>
uv pip install --upgrade <package>
```

---

## Python Version

- **Minimum**: Python 3.10 (required by `mcp` package)
- **System Python**: 3.9.6 (too old, don't use)
- **Available**: `/Users/moha/.local/bin/python3.10`

```bash
/Users/moha/.local/bin/python3.10 -m venv .venv
source .venv/bin/activate
uv pip install -e ".[dev]"
mitmproxy-mcp install-shims --force
```

---

## Project Structure

```
mitmproxy-mcp/
  addon.py                  thin wrapper (mitmproxy loads this)
  pyproject.toml            package config, dependencies
  pytest.ini                asyncio_mode=auto
  README.md                 user docs with client setup guides
  SKILL.md                  agent skill definition (agentskills.io format)
  LICENSE                   MIT
  .github/workflows/ci.yml  CI: ruff + mypy + pytest

  mitmproxy_mcp/            main package
    __init__.py             version = "0.1.0"
    __main__.py             CLI stub
    addon.py                mitmproxy addon, MCP server lifecycle
    models.py               pydantic models for flow serialization
    storage.py              thread-safe in-memory flow storage
    privacy.py              redaction engine (tokens, keys, passwords, JWTs)
    transport.py            stdio, sse, tcp transport layer
    tools/
      __init__.py
      flows.py              8 flow query/export tools
      replay.py             4 replay and modification tools
      intercept.py          5 interception control tools
      config.py             3 proxy configuration tools

  tests/                    174 tests, all passing
    conftest.py             shared fixtures
    test_models.py          model serialization tests
    test_flow_tools.py      flow query tool tests
    test_replay_tools.py    replay tool tests
    test_intercept_tools.py intercept tool tests
    test_config_tools.py    config tool tests
    test_privacy.py         redaction engine tests
    test_transport.py       transport layer tests
    test_integration.py     addon integration tests

```

---

## Running the Server

### Via config (recommended)

`~/.mitmproxy/config.yaml`:

```yaml
scripts:
  - /absolute/path/to/mitmproxy-mcp/addon.py

mcp_transport: sse
mcp_port: 9011
```

Then run mitmproxy normally:

```bash
mitmproxy      # interactive TUI
mitmweb        # web interface
mitmdump       # headless
```

### Via flags

```bash
mitmdump -s addon.py --set mcp_transport=sse --set mcp_port=9011
```

The MCP server starts automatically at `http://localhost:9011/sse`.

---

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `mcp_transport` | `stdio` | Transport: `stdio`, `sse`, or `tcp` |
| `mcp_port` | `9011` | Port for SSE/TCP transport |
| `mcp_max_flows` | `1000` | Max flows in memory (oldest evicted first) |
| `mcp_redact` | `false` | Enable privacy redaction of sensitive data |
| `mcp_redact_patterns` | _(empty)_ | Additional redaction patterns as JSON array (requires `mcp_redact: true`) |
| `mcp_view_sync_actions` | `all` | Which MCP actions sync to mitmproxy view: `all`, `none`, `replay`, `clear`, or `replay,clear` |

---

## Tools (20 total)

### Flow tools (8): `tools/flows.py`
get_flows, get_flow_by_id, search_flows, get_flow_request, get_flow_response, clear_flows, get_flow_count, export_flows

### Replay tools (4): `tools/replay.py`
replay_request, send_request, modify_and_send, duplicate_flow

### Intercept tools (5): `tools/intercept.py`
set_intercept_filter, get_intercepted_flows, resume_flow, resume_all, drop_flow

### Config tools (3): `tools/config.py`
get_options, set_option, get_status

---

## Coding Conventions

### Pydantic

We use Pydantic v2 but with the v1 compatibility API (`class Config` instead of `model_config`, `.dict()` instead of `.model_dump()`). This is because mitmproxy's internal pydantic usage expects v1 patterns.

```python
class MyModel(BaseModel):
    field: str

    class Config:
        arbitrary_types_allowed = True

model_dict = my_model.dict()
```

### mitmproxy Flow Structure

```python
flow.id                  # str: unique ID
flow.request.method      # str: "GET", "POST", etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moha-abdi/mitmproxy-mcp](https://github.com/moha-abdi/mitmproxy-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
