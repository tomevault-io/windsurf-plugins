---
trigger: always_on
description: - **Skills** = Claude Code native format (.claude/skills/ with SKILL.md) - Auto-discovered
---

# CLAUDE.md - Claude Code Operational Guide

**Terminology:**
- **Skills** = Claude Code native format (.claude/skills/ with SKILL.md) - Auto-discovered
- **Scripts** = CLI-based Python workflows (./scripts/) - Agent-agnostic

**Dual-Mode MCP pattern**: Reusable scripts (PREFERRED, 99.6% reduction) with CLI arguments, OR direct script writing (98.7% reduction) for novel tasks. Progressive disclosure via filesystem. Multi-transport support (stdio + SSE + HTTP).

## Execution Modes

### PRIMARY: Scripts-Based Execution (>2 tools, complex logic)

**When to use:**
- Multi-step research workflows
- Cross-validation needed
- Data processing pipelines
- Chaining multiple MCP servers
- 99.6% token reduction, 96% time reduction

**Pattern:**
1. `ls scripts/` - Discover available scripts
2. `cat scripts/{script}.py` - Read script docstring and CLI arguments
3. Execute with args (DO NOT edit file):
   ```bash
   # Example: Simple fetch
   uv run python -m runtime.harness scripts/simple_fetch.py \
       --url "https://example.com"

   # Example: Multi-tool pipeline
   uv run python -m runtime.harness scripts/multi_tool_pipeline.py \
       --repo-path "." \
       --max-commits 5
   ```

**Generic Example Scripts:**

Reusable CLI workflows (./scripts/):
- `simple_fetch.py` - Basic single-tool pattern (`--url`)
- `multi_tool_pipeline.py` - Multi-tool chaining pattern (`--repo-path`, `--max-commits`)

**Note:** These are **templates** - use as examples to create custom scripts for your specific MCP servers and use cases.

**Claude Code Users:** These scripts are also available as native Skills in `.claude/skills/` (SKILL.md format, auto-discovered).

### ALTERNATIVE: Direct Script Writing (1 tool, simple fetch)

**When to use:**
- Single tool call
- Straightforward data retrieval
- Novel workflows not covered by existing scripts
- Prototyping new patterns

**Pattern:** (existing documentation)
1. Explore `servers/` to discover tools
2. Write Python script using tool imports
3. Execute: `uv run python -m runtime.harness workspace/script.py`

## MCP Server Configuration

> **Important:** This project uses `mcp_config.json` in the project root, **separate from Claude Code's global MCP configuration** (`~/.claude.json`).
>
> To avoid conflicts:
> - Option A: Use different servers in each configuration
> - Option B: Disable overlapping servers in `~/.claude.json` while using this project

## Commands
- `uv run mcp-generate` - Gen Python wrappers from project's `mcp_config.json`
- `uv run mcp-discover` - Gen Pydantic types from actual API responses (see `discovery_config.json`)
- `uv run mcp-exec <script.py>` - Run script w/ MCP (direct or sandbox mode)
- `uv run mcp-exec <script> --args` - Run script with CLI arguments
- Example scripts: `workspace/example_progressive_disclosure.py`, `tests/integration/test_*.py`
- User scripts go in: `workspace/` (gitignored)

## Core Files
- `src/runtime/mcp_client.py` - `McpClientManager`: lazy loading, `initialize()` loads config only, `call_tool()` connects on-demand, tool format `"serverName__toolName"`, singleton via `get_mcp_client_manager()`
- `src/runtime/harness.py` - Exec harness: dual-mode (direct/sandbox), asyncio, MCP init, signal handlers, cleanup
- `src/runtime/generate_wrappers.py` - Auto-gen: connects all servers (stdio/SSE/HTTP), introspects schemas, generates `servers/<server>/<tool>.py` + `__init__.py`
- `src/runtime/discover_schemas.py` - Schema discovery: calls safe read-only tools, generates `servers/<server>/discovered_types.py` from real responses
- `src/runtime/normalize_fields.py` - Field normalization: auto-converts inconsistent API field casing (e.g., ADO: `system.parent` → `System.Parent`)
- `src/runtime/sandbox/` - Container sandboxing: rootless isolation, security controls, optional execution mode

## Structure
`servers/` (gitignored, regen w/ `uv run mcp-generate`):
```
servers/<serverName>/<toolName>.py         # Pydantic models, async wrapper
servers/<serverName>/__init__.py           # Barrel exports
servers/<serverName>/discovered_types.py   # Optional: Pydantic types from actual API responses
```

`scripts/` (CLI-based parameter templates - edit logic freely):
```
scripts/<script_name>.py                    # Workflow with argparse, USAGE docstring
scripts/README.md                           # Scripts documentation
scripts/SCRIPTS.md                          # Complete framework guide
```

`mcp_config.json` format (multi-transport):
```json
{
  "mcpServers": {
    "name_stdio": {
      "type": "stdio",
      "command": "command",
      "args": ["arg1"],
      "env": {}
    },
    "name_sse": {
      "type": "sse",
      "url": "https://...",
      "headers": {"Authorization": "Bearer ..."}
    },
    "name_http": {
      "type": "http",
      "url": "https://...",
      "headers": {"x-api-key": "..."}
    }
  },
  "sandbox": {
    "enabled": false,
    "runtime": "auto",
    "image": "python:3.11-slim"
  }
}
```

`discovery_config.json` format (optional, for schema discovery):
```json
{"servers": {"name": {"safeTools": {"tool_name": {"param1": "value"}}}}}
```

## Workflow

### Scripts-Based (PREFERRED)
1. Discover: `ls scripts/` → see available script templates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoloshii/mcp-code-execution-enhanced](https://github.com/yoloshii/mcp-code-execution-enhanced) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
