---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Packet Tracer MCP Server** — a Model Context Protocol server that enables LLMs to create, configure, validate, and **deploy in real-time** network topologies to Cisco Packet Tracer. The server provides **35 MCP tools** and 5 MCP resources, including live deployment via HTTP bridge, topology intelligence, config templates, and scenario presets.

**Tech Stack:** Python 3.11+, Pydantic 2.0+, MCP (FastMCP), Jinja2, Streamable HTTP
**Transport:** `http://127.0.0.1:39000/mcp` (streamable-http) | `--stdio` for legacy
**Version:** 0.5.0

## Common Commands

### Run the MCP Server
```bash
# Streamable HTTP on :39000 (default)
python -m src.packet_tracer_mcp

# Stdio transport (debug/legacy clients)
python -m src.packet_tracer_mcp --stdio

# Custom port via env var
PT_MCP_PORT=8080 python -m src.packet_tracer_mcp
```

### Install/Reinstall
```bash
# Production
pip install -e .

# Development (includes pytest, ruff, mypy)
pip install -e ".[dev]"
```

### Run Tests
```bash
# All tests (129 tests)
python -m pytest tests/ -v

# Single test file
python -m pytest tests/test_full_build.py -v

# Specific test
python -m pytest tests/test_full_build.py::TestFullBuild::test_basic_2_routers -v

# With coverage
python -m pytest tests/ --cov=src/packet_tracer_mcp --cov-report=term-missing
```

### Lint & Format
```bash
# Lint
ruff check src/

# Auto-fix
ruff check src/ --fix

# Format
ruff format src/

# Type check
mypy src/
```

### Configuration

**Claude Code** (`.mcp.json` in project root):
```json
{
  "mcpServers": {
    "packet-tracer": {
      "type": "http",
      "url": "http://127.0.0.1:39000/mcp"
    }
  }
}
```

**VS Code** (`.vscode/mcp.json`):
```json
{
  "servers": {
    "packet-tracer": {
      "url": "http://127.0.0.1:39000/mcp"
    }
  }
}
```

**Claude Desktop** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "packet-tracer": {
      "url": "http://127.0.0.1:39000/mcp"
    }
  }
}
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PT_MCP_PORT` | `39000` | Server HTTP port |
| `PT_MCP_LOG_LEVEL` | `INFO` | Logging level (DEBUG, INFO, WARNING, ERROR) |

## Architecture

This project follows **Clean Architecture / Domain-Driven Design** with clear layer separation:

```
src/packet_tracer_mcp/
├── adapters/mcp/          # MCP protocol layer
│   ├── tools/             # Split by domain concern (9 modules)
│   │   ├── catalog_tools.py       # pt_list_devices, pt_list_templates, pt_get_device_details
│   │   ├── planning_tools.py      # pt_estimate_plan, pt_plan_topology
│   │   ├── validation_tools.py    # pt_validate_plan, pt_fix_plan, pt_explain_plan,
│   │   │                          # pt_validate_config, pt_validate_topology
│   │   ├── generation_tools.py    # pt_generate_script, pt_generate_configs, pt_full_build
│   │   ├── deploy_tools.py        # pt_export, pt_deploy, pt_list_projects, pt_load_project,
│   │   │                          # pt_export_documentation
│   │   ├── bridge_tools.py        # pt_live_deploy, pt_bridge_status, pt_ping_bridge,
│   │   │                          # pt_undo_last_action, pt_load_last_plan,
│   │   │                          # pt_query/delete/rename/move/send_raw
│   │   ├── topology_tools.py      # pt_analyze_topology, pt_suggest_improvements,
│   │   │                          # pt_calculate_addressing
│   │   ├── preset_tools.py        # pt_list_presets, pt_load_preset
│   │   ├── template_tools.py      # pt_list_config_templates, pt_apply_template
│   │   └── _bridge_helpers.py     # HTTP helpers, bridge singleton, retry, persistence
│   ├── tool_registry.py  # Thin coordinator (delegates to tools/)
│   └── resource_registry.py
├── application/           # Use cases + DTOs (requests/responses)
├── domain/                # Core business logic
│   ├── models/           # TopologyPlan, DevicePlan, LinkPlan, errors, TopologyAnalysis
│   ├── services/         # Orchestrator, IPPlanner, Validator, AutoFixer, Explainer,
│   │                     # Estimator, TopologyAnalyzer, TemplateEngine, Presets
│   └── rules/            # Validation rules (devices, cables, IPs)
├── infrastructure/        # External concerns
│   ├── catalog/          # Device catalog, cable types, templates, aliases
│   ├── generator/        # PTBuilder (JS) + CLI config generators
│   ├── execution/        # HTTP bridge, live executor, deploy, manual export
│   └── persistence/      # Project repository (save/load)
├── shared/               # Enums, constants, utilities, logging
│   └── templates/        # 8 Jinja2 IOS config templates
├── server.py             # MCP server entry point
├── settings.py           # Server config (v0.5.0)
└── __main__.py           # python -m module entry
```

### Key Data Flow

1. **Request** → `TopologyRequest` (domain/models/requests.py)
2. **Orchestration** → `plan_from_request()` (domain/services/orchestrator.py)
3. **Validation** → `validate_plan()` (domain/services/validator.py)
4. **Generation** → PTBuilder script + CLI configs (infrastructure/generator/)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deiviidsito/MCP_Packet_Tracer](https://github.com/Deiviidsito/MCP_Packet_Tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
