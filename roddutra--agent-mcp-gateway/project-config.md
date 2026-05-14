---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent MCP Gateway is an MCP server that acts as a proxy/gateway to multiple downstream MCP servers. It enables per-agent/subagent access control, solving Claude Code's context window waste where all MCP tool definitions load upfront instead of being discovered on-demand.

**Core Problem:** When multiple MCP servers are configured, all tools from all servers (5,000-50,000+ tokens) load into every agent's context at startup. This wastes 80-95% of context on unused tools.

**Solution:** Gateway exposes only 3 minimal tools (~2k tokens) that allow agents to discover and request specific tools on-demand based on configurable access rules.

## Tech Stack

- **Python 3.12+** (required)
- **FastMCP 2.0** (version 2.13.0.1+) - MCP server framework
- **uv** - Package and project manager

## Development Commands

```bash
# Install dependencies
uv sync

# Run gateway
uv run python main.py

# Add dependency
uv add <package-name>

# Add dev dependency
uv add --dev <package-name>

# Update dependencies
uv lock --upgrade
```

See README.md for production MCP client configuration.

## Release Management

**CRITICAL:** NEVER bump version in `pyproject.toml` without explicit user approval.

**When version is updated:**
- Update ALL version files together: `pyproject.toml`, `server.json` (root `version` and `packages[0].version`), and `CHANGELOG.md`
- Run `uv lock` to sync lockfile
- DO NOT release without user approval

**To release:** See `docs/release-process.md` for complete workflow. Key: pushing the git tag (not just commits) triggers automated PyPI, MCP Registry, and GitHub release publishing via GitHub Actions.

## Architecture

### Gateway Model

```
Agent → Gateway (3 tools, ~2k tokens) → Policy Engine → Downstream MCP Servers (100s of tools)
         ↓
      Audit Log
```

**Traditional MCP:** All tools loaded upfront → Agent discovers what's available
**Gateway MCP:** Minimal interface loaded → Agent requests what it needs → Gateway provides filtered access

### Core Components

1. **Gateway Server** - FastMCP-based, exposes 3 gateway tools + 1 debug tool, uses `FastMCP.as_proxy()` for downstream proxying
2. **Policy Engine** - Stateless evaluation with deny-before-allow precedence, wildcard support
3. **Proxy Layer** - Transparent forwarding, stdio/HTTP transports, OAuth auto-detection
4. **Session Manager** - Per-agent isolation via FastMCP context

### Gateway Tools (Exposed to Agents)

```python
list_servers(agent_id: Optional[str], include_metadata: bool) -> List[Server]
get_server_tools(agent_id: Optional[str], server: str, names: Optional[List[str]],
                 pattern: Optional[str], max_schema_tokens: Optional[int]) -> List[Tool]
execute_tool(agent_id: Optional[str], server: str, tool: str,
             args: dict, timeout_ms: Optional[int]) -> Any
```

All tools accept optional `agent_id` (uses fallback chain if not provided). See README.md for detailed parameter descriptions.

### Configuration Structure

**MCP Servers Config** (`.mcp.json`):
```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-name"],
      "env": {"API_KEY": "${API_KEY}"}
    }
  }
}
```

**Gateway Rules Config** (`.mcp-gateway-rules.json`):
```json
{
  "agents": {
    "agent-name": {
      "allow": {"servers": ["server-name"], "tools": {"server-name": ["*"]}},
      "deny": {"tools": {"server-name": ["dangerous_*"]}}
    }
  },
  "defaults": {"deny_on_missing_agent": false}
}
```

See `config/*.example` files for complete examples and README.md for detailed configuration guide.

### Policy Evaluation Rules (CRITICAL - DO NOT CHANGE)

**Exact precedence order with short-circuit evaluation:**
1. Explicit deny rules → if match, DENY and STOP
2. Wildcard deny rules → if match, DENY and STOP
3. Explicit allow rules → if match, ALLOW and STOP
4. Wildcard allow rules → if match, ALLOW and STOP
5. Implicit grant - if server allowed but no tool rules specified for that server → ALLOW and STOP
6. Default policy → DENY

**Critical principle:** All deny rules (explicit + wildcard) checked before any allow rules.

**Implicit Grant Behavior:**
- If agent has server access AND no `allow.tools.{server}` entry → all tools from that server implicitly granted
- `allow.tools.{server}` entries are server-specific and narrow access for that server only
- `deny.tools.{server}` entries are server-specific and filter tools for that server only (evaluated in steps 1-2)

### Agent Identity

All gateway tools accept optional `agent_id` parameter. When not provided, uses fallback chain:

1. Explicit `agent_id` in tool call (highest priority)
2. `GATEWAY_DEFAULT_AGENT` env var
3. Agent named "default" in rules (if `deny_on_missing_agent` is false)
4. Error if none configured

See `docs/claude-code-subagent-mcp-limitations.md` for single-agent vs multi-agent configuration details.

### OAuth Support


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roddutra/agent-mcp-gateway](https://github.com/roddutra/agent-mcp-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
