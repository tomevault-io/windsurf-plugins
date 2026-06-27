---
trigger: always_on
description: This file is for autonomous agents or tooling that need to install, configure, verify, or modify `hiai-opencode`.
---

# AGENTS.md

This file is for autonomous agents or tooling that need to install, configure, verify, or modify `hiai-opencode`.

## Production Environment

**This is a production project.** All tools, MCP servers, skills, and integrations must be functional at all times. If any service stops working during a session (e.g., MemPalace MCP disconnects, build failures, test regressions), it must be diagnosed and fixed immediately — do not leave broken state behind.

## Purpose

`hiai-opencode` is an OpenCode plugin repository published as `@hiai-gg/hiai-opencode`. It is not a standalone app. It wires OpenCode agents, prompts, skills, MCP server launch commands, LSP defaults, and compatibility behavior around external upstream tools. Do not describe MCP servers, skills, model providers, or auxiliary OpenCode plugins as owned by this repository.

The normal workflow is:

1. install it through OpenCode or build it locally
2. register `@hiai-gg/hiai-opencode` in OpenCode
3. provide required environment variables
4. verify agent and MCP runtime state

## Install Workflow

### Native OpenCode Install

Use the OpenCode plugin CLI:

```bash
opencode plugin @hiai-gg/hiai-opencode@latest --global
```

Optional Dynamic Context Pruning plugin:

```bash
opencode plugin @tarquinen/opencode-dcp@latest --global
```

Manual alternative:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@hiai-gg/hiai-opencode"]
}
```

### Local Development Build

```bash
bun install
bun run build
```

### Register In OpenCode

Use the plugin CLI:

```bash
opencode plugin @hiai-gg/hiai-opencode@latest --global
```

Manual alternative:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@hiai-gg/hiai-opencode"]
}
```

### Verify

```bash
opencode debug config
hiai-opencode doctor
hiai-opencode mcp-status
hiai-opencode export-mcp .opencode/.mcp.json
opencode mcp list --print-logs --log-level INFO
```

`opencode mcp list` reads static `.opencode/.mcp.json` files in many OpenCode versions. Runtime MCP servers launched by the plugin may work but not appear there until a static config exists. The plugin auto-exports `.opencode/.mcp.json` on startup when missing. In `always` mode it overwrites only hiai-managed exports unless forced. `hiai-opencode export-mcp .opencode/.mcp.json` can refresh it manually.

## Post-Install Bootstrap Instructions

When a user asks OpenCode or another agent to finish installing this plugin, follow this procedure.

### Operating Rules

- Keep OpenCode plugins separate from MCP servers.
- Register `@hiai-gg/hiai-opencode` as an OpenCode plugin.
- Register `@tarquinen/opencode-dcp` only when the user wants Dynamic Context Pruning; it is a separate OpenCode plugin.
- Do not add MCP server npm packages to the OpenCode `plugin` array.
- Use `hiai-opencode.json` as the project-level service switchboard.
- Use `src/mcp/registry.ts` as the source of truth for default MCP launch wiring.
- Keep skill discovery deterministic by default: packaged plugin skills, generated builtin skills, explicit config sources, and project `.opencode/skills`.
- Do not enable global OpenCode, Claude, or Agents skill folders unless the user explicitly asks.
- Use `.env.example` as the key template, but never print, invent, commit, or hardcode secret values.
- Prefer user-level or project-local installs. Do not use sudo/admin rights unless the user explicitly asks.

### Bootstrap Checklist

1. Check tool availability:
   - `opencode --version`
   - `node --version`
   - `npx --version`
   - `bun --version`
   - `python --version` or `python3 --version`
   - `uv --version` when available
2. Check plugin registration with `opencode debug config`.
3. Find or create `hiai-opencode.json` in the project root or `.opencode/`.
4. Configure the `mcp` object there. Disable services that cannot run on the host.
5. Keep `skill_discovery` clean unless the user opts into external folders:
   - `config_sources: true`
   - `project_opencode: true`
   - `global_opencode: false`
   - `project_claude: false`
   - `global_claude: false`
   - `project_agents: false`
   - `global_agents: false`
6. Check environment variables without printing values:
   - `FIRECRAWL_API_KEY`
   - `STITCH_AI_API_KEY`
   - `CONTEXT7_API_KEY`
   - `MEMPALACE_PYTHON`
   
   - `HIAI_MCP_AUTO_INSTALL`
7. Verify with:
   - `opencode debug config`
   - `hiai-opencode doctor`
   - `hiai-opencode mcp-status`
   - `hiai-opencode export-mcp .opencode/.mcp.json` when static MCP visibility is needed
   - `opencode mcp list --print-logs --log-level INFO`

### MCP Setup Matrix

| Service | Enable when | Dependency behavior |
|---|---|---|
| `sequential-thinking` | Node and npx are available | Helper launcher runs `@modelcontextprotocol/server-sequential-thinking` |

| `mempalace` | `uv` is available, or Python 3.9+ with pip is available | Launcher prefers `uv`; otherwise uses Python and can run `python -m pip install --user mempalace` when `HIAI_MCP_AUTO_INSTALL` is not disabled. Interpreter can be pinned via `mcp.mempalace.pythonPath` or `MEMPALACE_PYTHON` |
| `stitch` | `STITCH_AI_API_KEY` is set | Remote MCP endpoint |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HiAi-gg/hiai-opencode](https://github.com/HiAi-gg/hiai-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
