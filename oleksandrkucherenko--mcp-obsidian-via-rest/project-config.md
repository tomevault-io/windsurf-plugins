---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides access to Obsidian vaults via the Local REST API plugin. The server can run as a standalone Node/Bun application or as a Docker container, enabling AI assistants to read, search, and interact with Obsidian notes.

**Multi-Transport Architecture:**
- **Transport Manager** (`src/transports/manager.ts`) - Manages lifecycle of multiple transport types
- **MCP Server Factory** (`src/server/mcp-server.ts`) - Creates MCP server instances with tools and resources
- **Transports:**
  - `stdio` - Standard input/output transport (default, best for local MCP clients)
  - `http` - HTTP JSON-RPC with streaming support (best for remote access)
- **Self-Healing API** (`src/api/self-healing.ts`) - Automatic URL selection and reconnection
- **Configuration System** (`src/config.ts`) - Multi-URL and transport configuration

## Development Commands

### Building and Running
```bash
# Install dependencies
bun install

# Build production bundle
bun run build

# Run locally (development mode with DEBUG logs)
bun run start:dev

# Run locally (production mode)
bun run start
```

### Testing
```bash
# Unit tests (src/**/*.test.ts)
bun test ./src

# E2E tests (requires Obsidian REST API running)
bun test:e2e

# Container tests (uses testcontainers)
bun test:containers
```

### Code Quality
```bash
# Type checking
bun run checks:types

# Linting (Biome)
bun run checks:lint

# Formatting (Biome)
bun run checks:format

# Unused code detection
bun run checks:knip
```

### Docker Operations
```bash
# Build local Docker image
bun run docker:latest

# Run Docker container (requires API_KEY and API_URLS env vars)
bun run docker:run

# Start E2E test environment (dockerized Obsidian)
bun run docker:e2e:start

# Stop E2E test environment
bun run docker:e2e:stop
```

### MCP Inspector (Debugging)
```bash
# Debug with local source
bun run mcp:inspector:local

# Debug with Docker container
bun run mcp:inspector:docker
```

### Publishing
```bash
# Prepare package for publishing
bun run publish:prepare

# Create release (uses release-it)
bun run release

# Dry-run release
bun run release:dry

# Create conventional commit
bun run commit
```

## Architecture Details

### Multi-Transport Architecture

The MCP server supports multiple transport types simultaneously. Each transport gets its own isolated MCP server instance with tools and resources registered.

**Transport Manager** (`src/transports/manager.ts`):
- Accepts a server factory function that creates new MCP server instances
- Creates a separate server instance for each enabled transport
- Manages lifecycle (start/stop) for all transports
- Provides status information for all transports

**Why Separate Server Instances?**
The MCP SDK's `server.connect(transport)` method replaces any existing transport. To support multiple transports simultaneously, each transport needs its own server instance.

**Architecture Diagram:**
```
┌─────────────────────────────────────────┐
│       Self-Healing Obsidian API         │
│  (Multi-URL selection & reconnection)    │
└──────────────────┬──────────────────────┘
                   │
        ┌──────────┴──────────┐
        │  Server Factory     │
        │  (creates instances)│
        └──────────┬──────────┘
                   │
            ┌──────┴──────┐
            │             │
      ┌─────▼───┐   ┌─────▼────┐
      │  MCP    │   │   MCP    │
      │ Server  │   │  Server  │
      │ (stdio) │   │  (http)  │
      └────┬────┘   └─────┬────┘
           │              │
      ┌────▼────┐   ┌─────▼──────┐
      │ Stdio   │   │   HTTP     │
      │Transport│   │ Transport  │
      └─────────┘   │ (Hono +    │
                    │  SSE)      │
                    └────────────┘
```

### MCP Server Implementation

The MCP server factory (`src/server/mcp-server.ts`) creates server instances that register:

**Tools:**
- `get_note_content` - Retrieves note content and metadata by file path
- `obsidian_search` - Searches notes using query strings
- `obsidian_semantic_search` - Semantic search (currently same as regular search)

**Resources:**
- `obsidian://{name}` - Resource template for accessing notes via URI (e.g., `obsidian://Skills/JavaScript/CORS.md`)

Each server instance is independent, so tools/resources are registered on each instance separately.

### HTTP Transport

**Hono-based HTTP Server** (`src/transports/http.transport.ts`):
- Uses `WebStandardStreamableHTTPServerTransport` for full MCP protocol support
- Supports JSON-RPC over HTTP POST
- Supports SSE streaming for responses
- Includes `/health` endpoint for health checks
- Optional Bearer token authentication via `MCP_HTTP_TOKEN`

**Request Logger** (`src/transports/hono-logger.ts`):
- Logs incoming/outgoing HTTP requests
- Uses `debug` library (writes to stderr, not stdout)
- Inspired by Hono's built-in logger middleware

### Self-Healing API

**Self-Healing Wrapper** (`src/api/self-healing.ts`):
- Tests multiple URLs in parallel on startup
- Selects fastest responding URL
- Monitors connection health every 30 seconds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OleksandrKucherenko/mcp-obsidian-via-rest](https://github.com/OleksandrKucherenko/mcp-obsidian-via-rest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
