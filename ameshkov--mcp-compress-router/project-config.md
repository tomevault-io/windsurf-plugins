---
trigger: always_on
description: MCP Compress Router — a single-router MCP server that compresses all
---

# AGENTS.md

MCP Compress Router — a single-router MCP server that compresses all
connected MCP servers into one, with just two tools: `get_tool_schema` and
`invoke_tool`. Saves up to 99% on token overhead by replacing verbose
tool listings with a compact routing layer.

## Table of Contents

- [Project Overview](#project-overview)
- [Technical Context](#technical-context)
- [Project Structure](#project-structure)
- [Build and Test Commands](#build-and-test-commands)
- [Contribution Instructions](#contribution-instructions)
- [Code Guidelines](#code-guidelines)
    - [Architecture](#architecture)
    - [Code Quality](#code-quality)
    - [Testing](#testing)
    - [Dependency Management](#dependency-management)
    - [Configuration & Documentation](#configuration--documentation)
    - [Markdown Formatting](#markdown-formatting)

## Project Overview

A single MCP (Model Context Protocol) server that acts as a router. Instead
of sending all tool names and descriptions from every connected MCP to the
LLM on every request, this server exposes only two tools:

- **`get_tool_schema`** — returns the JSON parameter schema for one or more
  tools on a connected MCP server.
- **`invoke_tool`** — forwards a tool invocation to a connected MCP server
  and returns the result.

The LLM first calls `get_tool_schema` to learn the parameters, then calls
`invoke_tool` to execute. This reduces token overhead by ~96% for a typical
coding session with 3 MCP servers.

## Technical Context

| Field | Value |
| --- | --- |
| Language | TypeScript 5.9, ES2022 target, strict mode |
| Runtime | Node.js 24+ |
| Package Manager | pnpm 10+ |
| Framework | MCP SDK (`@modelcontextprotocol/sdk`) |
| Linting | oxlint (category-based config) + Knip |
| Formatting | Prettier 3.x, Markdownlint (markdownlint-cli2) |
| Project Type | MCP server (stdio transport) |

## Project Structure

```text
mcp-compress-router/
├── src/                      # Application source code
│   ├── index.ts              # MCP server entry point (stdio transport) + CLI dispatch
│   ├── cli/                   # Management CLI subcommands
│   │   ├── index.ts           # Barrel exports (public API)
│   │   ├── config-io.ts       # Raw mcp.json read/write with first-use creation
│   │   ├── add-command.ts     # add subcommand handler
│   │   ├── disable-command.ts # disable subcommand handler
│   │   ├── enable-command.ts # enable subcommand handler
│   │   ├── remove-command.ts  # remove subcommand handler
│   │   ├── get-command.ts     # get subcommand handler
│   │   ├── list-command.ts    # list subcommand handler
│   │   ├── tools-command.ts   # tools subcommand handler (live inspection)
│   │   ├── login-command.ts   # login subcommand handler (OAuth flow)
│   │   ├── logout-command.ts  # logout subcommand handler (clear credentials)
│   │   ├── register-commands.ts # Wires all CLI subcommands onto a commander program
│   │   └── router-runner.ts   # Router startup: connect servers, build catalog, serve, and shut down
│   ├── services/             # Core business logic
│   │   ├── index.ts           # Barrel exports (public API)
│   │   ├── config.ts          # Configuration loader
│   │   ├── discovery.ts       # Downstream server discovery (single-server connect + tool listing)
│   │   ├── catalog.ts         # Catalog Builder & Cache
│   │   ├── server-connection.ts # Per-server client lifecycle (connect, reconnect, invoke, close)
│   │   ├── invoke-with-recovery.ts # Self-recovery orchestration on invoke_tool
│   │   ├── guided-error.ts    # Detailed guided error message builder
│   │   ├── auth-errors.ts     # GuidedAuthError tagged error class
│   │   ├── tool-cache.ts      # Disk cache for tool schemas (tools-cache.json)
│   │   ├── oauth.ts           # OAuth credential storage (credentials.json) + proactive refresh & invalidation
│   │   ├── auth-status.ts     # OAuth requirement probe & auth-status lookup
│   │   ├── oauth-discovery.ts # Spec-compliant two-step OAuth discovery (PRM -> AS)
│   │   ├── shutdown-coordinator.ts # Graceful shutdown orchestration (run cleanup hooks once)
│   │   └── shutdown-triggers.ts # Signal & stdin-EOF triggers that start a shutdown
│   ├── utils/                 # Shared utilities
│   │   ├── index.ts           # Barrel exports (public API)
│   │   ├── expand-env.ts      # ${VAR} / ${VAR:-default} expansion
│   │   ├── argument-names.ts  # Argument Name Extractor (inputSchema.properties keys)
│   │   ├── description-truncator.ts # Description Truncator (medium-level first-sentence snippet)
│   │   ├── compression-level.ts # CompressionLevel valid set + type guard
│   │   ├── parse-jsonc.ts     # JSONC parser wrapper (comments + trailing commas)
│   │   ├── text-format.ts     # Compact catalog text renderer
│   │   ├── tool-filter.ts     # Tool Filter (allow/deny glob matching)
│   │   ├── types.ts           # Shared type definitions
│   │   ├── validate-arguments.ts # JSON Schema argument validation
│   │   ├── validate-glob.ts   # Glob pattern validator
│   │   ├── timeout.ts         # Downstream/discovery timeout budgets + timeout fetch
│   │   ├── logger.ts          # Level-aware structured logger
│   │   └── open-browser.ts    # Platform-safe browser opener using spawn()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ameshkov/mcp-compress-router](https://github.com/ameshkov/mcp-compress-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
