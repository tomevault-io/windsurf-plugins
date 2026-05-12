---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference Commands

```bash
# Run full test suite
./run_tests.sh
# Or directly:
python3 test_agent.py --output test_report.json

# Generate remediation report from test failures
python3 remediate.py test_report.json

# Verify all MCP tools have test coverage
python3 verify_test_coverage.py

# Run server locally (for debugging/development - MCP clients auto-start the server)
uv run jamf-mcp

# Run single test (manually invoke specific test method)
python3 -c "
import asyncio
from test_agent import JamfMCPTestAgent
agent = JamfMCPTestAgent()
asyncio.run(agent.test_get_computers_list())
"
```

## Architecture Overview

This is an MCP (Model Context Protocol) server that enables LLMs to interact with Jamf's ecosystem for Apple device management and security.

### Supported Products

| Product | Description | Tools Available |
|---------|-------------|-----------------|
| **Jamf Pro** | Core device management for macOS, iOS/iPadOS, and tvOS | 40 tools |
| **Jamf Protect** | Endpoint security for threat detection and response | 6 tools |
| **Jamf Security Cloud** | Device risk management via RISK API | 2 tools |
| **Setup** | Zero-credential onboarding (always available) | 2 tools |

### Core Flow

```
Claude (MCP Client) → FastMCP Server (server.py) → Tool Functions (tools/*.py) → Client → Jamf API

Product-Specific Clients:
- JamfClient (client.py) → Jamf Pro API
- ProtectClient (protect_client.py) → Jamf Protect API
- JamfSecurityClient (security_client.py) → Jamf Security Cloud API
```

### Key Components

| Component | Location | Purpose |
|-----------|----------|---------|
| MCP Server | `src/jamf_mcp/server.py` | FastMCP entry point, lifespan management, client initialization |
| Tool Registry | `src/jamf_mcp/tools/_registry.py` | Decorator system (`@jamf_tool`) for automatic tool registration |
| Tool Modules | `src/jamf_mcp/tools/*.py` | Domain-specific tool implementations |
| Jamf Pro Client | `src/jamf_mcp/client.py` | HTTP client for Jamf Pro (Classic, v1, v2, v3 APIs) |
| Jamf Pro Auth | `src/jamf_mcp/auth.py` | OAuth client credentials flow for Jamf Pro |
| Protect Client | `src/jamf_mcp/protect_client.py` | HTTP client for Jamf Protect API |
| Protect Auth | `src/jamf_mcp/protect_auth.py` | Basic auth for Jamf Protect |
| Security Client | `src/jamf_mcp/security_client.py` | HTTP client for Jamf Security Cloud (RISK API) |
| Security Auth | `src/jamf_mcp/security_auth.py` | Bearer token auth for Security Cloud |
| Test Agent | `test_agent.py` | Integration tests against live Jamf instances |
| Coverage Verification | `verify_test_coverage.py` | Ensures all tools have corresponding tests |
| Auto-Remediation | `remediate.py` | Generates remediation reports from test failures |

### Jamf Pro API Architecture

The Jamf Pro API uses different versions for different resources:

| API | Endpoint Pattern | Format | Used For |
|-----|------------------|--------|----------|
| Classic | `/JSSResource/...` | XML for POST/PUT, JSON for GET | Users, groups, policies, profiles |
| v1 | `/api/v1/...` | JSON | Computers, scripts, categories, app installers |
| v2 | `/api/v2/...` | JSON | Mobile devices, mobile device prestages |
| v3 | `/api/v3/...` | JSON | Computer prestages |

The `JamfClient` class in `client.py` provides methods for each API version:
- Classic: `classic_get()`, `classic_post()`, `classic_put()`, `classic_delete()`
- Modern APIs: `v1_get()`, `v1_post()`, `v1_put()`, `v1_patch()`, `v1_delete()`, `v2_get()`, etc.

### Tool Registration System

Tools are registered using the `@jamf_tool` decorator from `_registry.py`:

```python
from ._registry import jamf_tool

@jamf_tool
async def jamf_your_tool_name(param: str) -> str:
    """Docstring becomes the MCP tool description."""
    client = get_client()
    result = await client.v1_get("/endpoint")
    return format_response(result, "Success message")
```

The decorator automatically collects all tools, and `register_all_tools()` in `server.py` registers them with FastMCP during server startup.

## Critical Rules

### After Any Code Changes

Run tests before considering work complete:
```bash
./run_tests.sh
```

This script runs the test suite and automatically generates a remediation report (`remediation_report.json`) if tests fail. The report maps failed tests to the source files that need fixing.

### Adding a New MCP Tool

1. Add implementation in appropriate `src/jamf_mcp/tools/<module>.py`
2. Export in `src/jamf_mcp/tools/__init__.py`
3. Register with `@mcp.tool()` in `src/jamf_mcp/server.py`
4. Add test in `test_agent.py` and include in `run_all_tests()` test_plan
5. Add tool-to-test mapping in `TOOL_TEST_MAPPING` in `verify_test_coverage.py`
6. Update README.md "Available Tools" section
7. Add required Jamf Pro API privileges to `docs/INSTALLATION.md` (use exact privilege names from Jamf Pro, e.g., "Read Computers", "Create Smart Computer Groups")

### Removing a Tool

Reverse the above steps - remove from all listed files.

### Looking Up Jamf API Endpoints

Source of truth: `https://developer.jamf.com/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jamf-Concepts/mcp-hub](https://github.com/Jamf-Concepts/mcp-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
