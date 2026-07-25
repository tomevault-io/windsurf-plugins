---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Development
- `npm run build` - Full build process including code generation and TypeScript compilation (recommended for development)
- `npm run dev` - Start MCP inspector for debugging (`@modelcontextprotocol/inspector`)
- `make build` - Docker-based build that copies both `dist/` and `td/modules/` from container (for CI/CD)
- `npm run build:mcpb` - Build MCP Bundle package for distribution
- `make clean` - Remove all generated files (`dist`, `td/modules`, `node_modules`)

### Code Generation Workflow
The project uses OpenAPI 3.0.0 schema-based code generation with a three-step process:

- `npm run gen:openapi` - Bundle OpenAPI schema files into single YAML using `@redocly/cli`
- `npm run gen:handlers` - Generate Python handlers using custom Node.js script with Mustache templates
- `npm run gen:mcp` - Generate TypeScript client code and Zod schemas using Orval v8
- `npm run gen` - Run all generation steps in sequence

### Testing and Quality
- `npm test` - Run all tests (integration and unit)
- `npm run test:integration` - Integration tests with TouchDesigner WebServer
- `npm run test:unit` - Unit tests for MCP server components
- `npm run coverage` - Generate test coverage report

### HTTP Transport Mode
- `npm run http` - Build and start the MCP server in HTTP mode (port 6280, TD on 9981)
- `npm run test:integration` - Includes the HTTP transport suite (`tests/integration/httpTransport.test.ts`)

**HTTP Mode Configuration:**
- Default port: `3000`
- Default host: `127.0.0.1`
- Endpoint: `/mcp`
- Health check: `GET /health`

### Linting and Formatting

The project uses multiple formatters and linters for different languages:

**All languages:**

- `npm run lint` - Run all linters (Biome, TypeScript, Ruff, Prettier)
- `npm run format` - Auto-fix formatting for all languages

**TypeScript/JavaScript (Biome):**

- `npm run lint:biome` - Lint TypeScript/JavaScript files
- `npm run format:biome` - Format and fix TypeScript/JavaScript files
- Sorts imports and object keys automatically via Biome Assist

**Python (Ruff):**

- `npm run lint:python` - Lint Python files in `td/` directory
- `npm run format:python` - Format and fix Python files (includes import sorting)
- Configuration in `pyproject.toml`
- Note: Only `td/modules/td_server/openapi_server/openapi/openapi.yaml` is auto-generated

**YAML (Prettier):**

- `npm run lint:yaml` - Check YAML file formatting
- `npm run format:yaml` - Format YAML files (uses Prettier)
- Configuration in `.prettierrc.json`

**Important:**

- Python (Ruff) must be installed separately via pip/pipx/uv for formatting to work

## Architecture Overview

### Dual-Process Architecture
This MCP server operates as a bridge between AI agents and TouchDesigner through a dual-process architecture:

1. **Node.js MCP Server** (`src/`) - Implements MCP protocol, handles AI agent communication
2. **Python WebServer** (`td/modules/`) - Runs inside TouchDesigner via WebServer DAT, controls TD directly

Communication flows: AI Agent ↔ Node.js MCP Server ↔ HTTP API ↔ Python WebServer (in TouchDesigner)

### Key Components

#### MCP Server (Node.js)
- `TouchDesignerServer` class in `src/server/touchDesignerServer.ts` - Main MCP server implementation
- `TouchDesignerClient` in `src/tdClient/` - HTTP client for communicating with TD WebServer
- Tool definitions in `src/features/tools/toolDefinitions.ts` - `TOOL_DEFINITIONS` is the single source of truth for each MCP tool (name, description, input schema, handler). `handlers/tdTools.ts` registers them in a loop, and the `describe_td_tools` manifest derives its parameter metadata from each tool's Zod schema via introspection
- Code generation outputs in `src/gen/` - Auto-generated API client and Zod schemas

#### TouchDesigner Integration (Python)
- `mcp_webserver_base.tox` - Main TouchDesigner component to import
- `api_controller.py` - Routes HTTP requests using OpenAPI schema
- `api_service.py` - Business logic for TouchDesigner operations
- `generated_handlers.py` - Auto-generated handler stubs (connects controller to service)

### Transport Architecture

The transport layer uses a factory + manager pattern to support stdio and streamable HTTP modes:

- `src/transport/factory.ts` – Validates transport config (stdio vs HTTP) and instantiates the appropriate MCP transport
- `src/transport/expressHttpManager.ts` – Wraps `StreamableHTTPServerTransport` inside an Express server, wiring `/mcp` and `/health`, plus graceful shutdown
- `src/transport/sessionManager.ts` – Tracks HTTP sessions (UUIDs, TTL cleanup) for health metrics and future SDK callbacks
- `src/transport/config.ts` – Type definitions and Zod validators for `TransportConfig`

Design references: `.doc/streamable-http-implementation-plan.md` and `.doc/refactor_sdk_first.md` cover the SDK-first approach and HTTP rollout plan.

### Code Generation System
The project uses OpenAPI 3.0.0 schema (`src/api/index.yml`) for maintaining consistency:

- OpenAPI schema bundled via `@redocly/cli` to `td/modules/td_server/openapi_server/openapi/openapi.yaml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [8beeeaaat/touchdesigner-mcp](https://github.com/8beeeaaat/touchdesigner-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
