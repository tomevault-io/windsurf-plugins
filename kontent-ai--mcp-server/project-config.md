---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development Commands
```bash
# Install dependencies
npm ci

# Build the project (TypeScript compilation)
npm run build

# Start development server with auto-reload (no build required)
npm run dev:stdio   # For STDIO transport
npm run dev:shttp   # For Streamable HTTP transport

# Start production server (requires build)
npm run start:stdio  # For STDIO transport
npm run start:shttp  # For Streamable HTTP transport
```

### Code Quality Commands
```bash
# Run formatter and linter check
npm run format

# Auto-fix formatting and linting issues
npm run format:fix
```

### Debugging
```bash
# Debug with MCP inspector
npx @modelcontextprotocol/inspector -e KONTENT_API_KEY=<key> -e KONTENT_ENVIRONMENT_ID=<env-id> node build/bin.js

# Or inspect streamable HTTP server
npx @modelcontextprotocol/inspector
```

## Architecture Overview

This is a Model Context Protocol (MCP) server for Kontent.ai that enables AI models to interact with Kontent.ai's APIs through natural language. The project follows a modular architecture:

### Core Components

1. **Transport Layer** (`src/bin.ts`): Single entry point supporting two transport protocols:
   - STDIO: Direct process communication (single-tenant only)
   - Streamable HTTP: Request-response based HTTP communication (supports multi-tenant)

2. **Server Core** (`src/server.ts`): Central server instance that:
   - Registers all available tools
   - Manages MCP server lifecycle
   - Coordinates tool execution

3. **Tools Directory** (`src/tools/`): Each tool is a separate module that:
   - Implements a specific Kontent.ai operation
   - Uses standardized error handling via `errorHandler.ts`
   - Returns responses using `createMcpToolSuccessResponse`
   - Must call `get-patch-guide` before any patch operation

4. **API Clients** (`src/clients/kontentClients.ts`): Manages Kontent.ai SDK instances:
   - Management API client for content operations
   - Includes source tracking headers for API usage analytics

5. **Validation Schemas** (`src/schemas/`): Zod schemas for input validation:
   - Content item, content type, taxonomy schemas
   - Specialized patch operation schemas
   - Workflow and variant filtering schemas

### Critical Development Rules

#### Tool Naming Conventions
Tools follow strict naming patterns enforced by Cursor rules:
- Format: `[action]-[entity]`
- Use full entity names: `content-type`, `content-type-snippet`, `content-item`, `content-item-variant`, `taxonomy-group`
- Example: `get-content-type`, `list-content-item-variants`, `get-content-type-snippet`

#### Tool Descriptions
Tool descriptions must follow a standardized pattern (enforced in `.cursor/rules/kontent-tool-descriptions.mdc`):
- Pattern: `"[Action] [Kontent.ai entity] [method/context]"`
- **Always include "Kontent.ai"** explicitly
- Example: "Retrieve Kontent.ai content type by ID"

#### README Synchronization
When modifying tools (enforced in `.cursor/rules/tools-in-readme.mdc`):
- **Adding tools**: Always describe them in README.md
- **Modifying tools**: Adjust descriptions in README.md accordingly
- **Removing tools**: Remove all mentions from README.md
- **Table of Contents**: Must contain only second-level headings (enforced in `.cursor/rules/toc-readme.mdc`)

### Environment Requirements

#### Transport-to-Mode Mapping
- **STDIO** = Single-tenant (credentials via env vars, local process communication)
- **Streamable HTTP** = Multi-tenant (credentials via Bearer token per request)

#### Single-Tenant Mode (STDIO)
Required environment variables:
- `KONTENT_API_KEY`: Management API key
- `KONTENT_ENVIRONMENT_ID`: Environment ID

Optional telemetry and configuration variables:
- `appInsightsConnectionString`: Application Insights connection string for telemetry
- `projectLocation`: Project location identifier for telemetry tracking
- `manageApiUrl`: Custom Management API base URL (e.g., for preview environments)

#### Multi-Tenant Mode (Streamable HTTP)
No credential environment variables required. Instead:
- Environment ID is provided via URL path: `/{environmentId}/mcp`
- API key is provided via Bearer token: `Authorization: Bearer <api-key>`
- `PORT`: Server port (optional, defaults to 3001)

##### Client Configuration Examples

**VS Code**: Create `.vscode/mcp.json` in your workspace:
```json
{
  "servers": {
    "kontent-ai-multi": {
      "uri": "http://localhost:3001/{environmentId}/mcp",
      "headers": {
        "Authorization": "Bearer {api-key}"
      }
    }
  }
}
```

**Claude Desktop**: Use `mcp-remote` as proxy in `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "kontent-ai-multi": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://localhost:3001/{environmentId}/mcp",
        "--header",
        "Authorization: Bearer {api-key}"
      ]
    }
  }
}
```

**Claude Code**: Configure via CLI:
```bash
claude mcp add --transport http kontent-ai-multi \
  "http://localhost:3001/{environmentId}/mcp" \
  --header "Authorization: Bearer {api-key}"
```

### Code Style

- TypeScript with ES2022 target, NodeNext modules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kontent-ai/mcp-server](https://github.com/kontent-ai/mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
