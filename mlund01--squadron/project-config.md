---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
go build -o squadron ./cmd/cli              # Build the CLI
./squadron init                            # Initialize encrypted vault
./squadron verify <path>                   # Validate HCL config
./squadron chat -c <path> <agent_name>     # Start chat with an agent
./squadron mission -c <path> <mission>     # Run a mission
./squadron mission -c <path> -d <mission>  # Run with debug logging
./squadron mission --resume <id> -c <path> <mission> # Resume a failed mission
./squadron vars set <name> <value>         # Set a variable
./squadron vars get <name>                 # Get a variable
./squadron vars list                       # List all variables
./squadron serve -c <path>                 # Connect to command center (requires command_center block)
./squadron serve -c <path> -w              # Launch local command center + connect
./squadron serve -c <path> -w --cc-port 9090  # Custom command center port
./squadron serve -c <path> -w --no-browser # Launch without opening browser
./squadron mcp status                      # Show OAuth status for configured MCP servers
./squadron mcp login <name>                # Authorize an MCP server via OAuth
./squadron mcp logout <name>               # Forget stored OAuth token for an MCP server
./squadron upgrade                         # Upgrade to latest release
./squadron upgrade --version v0.0.13       # Upgrade to specific version
./squadron version                         # Print current version
./squadron docs [output-dir]               # Extract docs to local folder
./squadron plugin tools <path>             # List plugin tools
./squadron plugin call <path> <tool> <json># Call a plugin tool
./squadron plugin info <path> <tool>       # Get plugin tool info
./squadron plugin build <source>           # Build a plugin from source
```

## Architecture Overview

Squadron is a declarative framework for building and running AI agent workflows. LLM-powered agents, tools, plugins, and multi-step missions are defined entirely in HCL configuration files — no code required. Agents reason and act autonomously using a two-tier architecture: commanders orchestrate tasks while agents execute tool calls via native function calling.

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `agent/` | Agent and Commander implementations, orchestration |
| `aitools/` | Tool interface, schema definitions, result interception |
| `config/` | HCL config loading with staged evaluation |
| `llm/` | LLM provider abstraction (Anthropic, OpenAI, Gemini, Ollama) |
| `plugin/` | gRPC plugin system using hashicorp/go-plugin |
| `mission/` | Mission runner, task execution, knowledge store |
| `store/` | Persistence interfaces and SQLite implementation |
| `scheduler/` | Cron-based mission scheduling and next-fire calculation |
| `streamers/` | Output streaming interfaces for CLI/TUI |
| `wsbridge/` | WebSocket bridge client for command center communication |
| `mcp/` | Consumer-side MCP client: loads external MCP servers (stdio/http/npm/github) declared in `mcp "name" { ... }` blocks and exposes their tools |
| `mcphost/` | Host-side MCP server: exposes Squadron's own tools over MCP when `mcp_host { ... }` is enabled |
| `internal/release/` | Shared GitHub-release download/extract helpers used by both plugin and MCP auto-install |
| `cmd/` | CLI commands and plugin entry points |

---

## HCL Config System (config/)

The config loading uses **staged evaluation** to support HCL expression references:

1. **Stage 1**: Load `variable` blocks (no context needed)
2. **Stage 1.4**: Load `packet` blocks with `vars` context. Done before every downstream stage so the HCL-exclusion filter (drops `.hcl` files inside any packet path) runs before vault / storage / command_center / mcp_host iterate `allParsedBlocks`.
3. **Stage 1.5**: Load `plugin` blocks and `mcp "name"` blocks with `vars` context. Both happen in the same stage because both expose tools through HCL namespaces that later stages need to resolve against.
4. **Stage 2**: Load `model` blocks with `vars` + `plugins` + `mcp` context → enables `api_key = vars.anthropic_api_key`
5. **Stage 3**: Load custom `tool` blocks with `vars` + `models` + `builtins` + `plugins` + `mcp` context → enables `implements = builtins.http.get`
6. **Stage 4**: Load `agent` blocks with `vars` + `models` + `tools` + `plugins` + `mcp` context → enables `tools = [plugins.playwright.all, mcp.filesystem.read_file, tools.weather]`
7. **Stage 5**: Load `mission` blocks with full context

Each stage uses partial structs with `hcl:",remain"` to ignore unknown block types during that pass.

### HCL Config Format

```hcl
# variables.hcl
variable "anthropic_api_key" {
  secret = true
}

# models.hcl
model "anthropic" {
  provider = "anthropic"
  api_key  = vars.anthropic_api_key
}

# plugins.hcl
plugin "playwright" {
  version = "local"
}

# mcp_servers.hcl — consumer-side MCP servers (Squadron pulling tools from external servers)
#
# Exactly one of `command`, `url`, or `source` per block.

# Auto-installed npm package (cache at .squadron/mcp/filesystem/2024.12.1/)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlund01/squadron](https://github.com/mlund01/squadron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
