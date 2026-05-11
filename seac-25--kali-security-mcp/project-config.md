---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**MCP-Kali-Server** is an MCP (Model Context Protocol) server that bridges AI agents with 200+ Kali Linux security tools. It supports penetration testing, CTF competitions, and security assessments through intelligent tool orchestration.

**Architecture**: v6.0 modular architecture. The system runs in **local execution mode** — `mcp_server.py` calls Kali tools directly via `subprocess`, no backend server needed. Core components: declarative `tool_registry`, structured `output_parsers`, `hybrid_decision_engine` for multi-agent routing, and session TTL auto-cleanup.

## Running the System

```bash
# Start MCP server (local execution mode, default)
python mcp_server.py

# Or use the startup script (creates venv if needed, checks tool availability)
bash start.sh

# With a specific tool profile
python mcp_server.py --tool-profile strict    # minimal tools
python mcp_server.py --tool-profile compliance # default, most tools
python mcp_server.py --tool-profile full       # all tools

# Force-enable/disable specific modules
python mcp_server.py --enable-module pwn --disable-module apt

# SSE mode for remote access
python mcp_server.py --transport sse --host 0.0.0.0 --port 8765

# Check system status
python status_check.py
```

The MCP configuration is in `.mcp.json`. Virtual environment is at `.venv/`.

## Installing Dependencies

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Optional: PWN tools
pip install pwntools
```

## Testing

```bash
pytest                         # run all tests
pytest -x                      # stop on first failure
pytest --cov=kali_mcp          # with coverage
pytest -k "test_name"          # run specific test
```

## Core Architecture

```
mcp_server.py (520 lines, entry point)
│   Creates FastMCP instance, initializes multi-agent system,
│   registers all tools via register_xxx_tools() pattern
│
├── kali_mcp/core/              Core logic (15+ modules)
│   ├── local_executor.py       LocalCommandExecutor — subprocess tool execution
│   ├── mcp_session.py          SessionContext + StrategyEngine
│   ├── agent_registry.py       Multi-agent registry (v4.0)
│   ├── agent_coordinator.py    Central coordinator agent
│   └── ...                     AI context, ML optimizer, knowledge graph, etc.
│
├── kali_mcp/mcp_tools/         MCP tool registration (28 modules, 200+ tools)
│   Each module exports register_xxx_tools(mcp, executor, ...) function
│
├── kali_mcp/agents/            17 specialized agents
│   ├── information_gathering/  ReconAgent, SubdomainAgent, WebReconAgent
│   ├── vulnerability_discovery/ VulnScanner, WebVuln, Auth, NetworkVuln, VulnVerifier
│   ├── exploitation/           ExploitAgent, PrivilegeAgent, LateralAgent
│   └── specialized/            PwnAgent, CryptoAgent, ForensicsAgent, CodeAudit, etc.
│
├── kali_mcp/security/          Authorization & compliance
│   ├── engagement.py           EngagementContext — scope/authorization management
│   └── tool_profile.py         Tool profiles: strict | compliance | full
│
├── kali_mcp/diggers/           Deep vulnerability diggers (v3.0)
├── deep_test_engine/           HTTP/WebSocket/gRPC interaction engine
└── pwnpasi/                    PWN automation (ROP, heap exploit, fuzzing)
```

### Request Flow

```
Claude AI → MCP protocol (stdio/SSE)
  → FastMCP instance (mcp_server.py)
    → Tool function (kali_mcp/mcp_tools/*.py)
      → LocalCommandExecutor.execute_tool_with_data()
        → Compliance check (EngagementManager + PentestPolicy)
          → Parameter sanitization (shlex.quote)
            → subprocess.run() executes Kali tool
              → Result returned to Claude
```

### Tool Registration Pattern (how to add new tools)

1. **Create or edit a module** in `kali_mcp/mcp_tools/`:

```python
# kali_mcp/mcp_tools/my_tools.py
def register_my_tools(mcp, executor):
    @mcp.tool()
    def my_new_tool(target: str, option: str = "default") -> Dict[str, Any]:
        """Tool description shown to the AI."""
        data = {"target": target, "option": option}
        return executor.execute_tool_with_data("toolname", data)
```

2. **Export from `__init__.py`**: Add `from .my_tools import register_my_tools` to `kali_mcp/mcp_tools/__init__.py`

3. **Register in `mcp_server.py`**: Add the import and call `_safe_register("my_module", "My Tools", register_my_tools, mcp, executor)` in `setup_mcp_server()`

4. **Add module key** to `ALL_MODULE_KEYS` in `kali_mcp/security/tool_profile.py` and configure which profiles enable it

5. **If using a new CLI tool**, add it to `ALLOWED_TOOLS` set in `kali_mcp/core/local_executor.py` (whitelist of 67 permitted binaries)

### Security Model

- **Tool whitelist**: `ALLOWED_TOOLS` in `local_executor.py` — only these binaries can be invoked
- **Parameter sanitization**: All arguments pass through `shlex.quote()` before subprocess execution
- **Tool profiles** (set via `--tool-profile` or `KALI_MCP_TOOL_PROFILE` env var):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeaC-25/Kali-Security-MCP](https://github.com/SeaC-25/Kali-Security-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
