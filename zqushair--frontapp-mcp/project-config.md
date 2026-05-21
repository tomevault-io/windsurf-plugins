---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Frontapp MCP Server** - a Model Context Protocol server that enables Claude Code to interact with the Frontapp API. It provides comprehensive access to conversations, messages, contacts, teammates, tags, inboxes, comments, and analytics.

The server runs in a Docker container and communicates with Claude Code via stdio (standard input/output).

## Development Commands

### Building and Running

```bash
# Build TypeScript
npm run build

# Development with watch mode
npm run watch

# Start the server (requires FRONTAPP_API_TOKEN env var)
npm start

# Or use Docker (recommended)
make build    # Build Docker image
make up       # Start container
make down     # Stop container
make restart  # Restart container
make logs     # View logs
make shell    # Open shell in container
```

### Docker Development

The project uses Docker Compose with a multi-stage build:
- Stage 1: Build TypeScript with full dependencies
- Stage 2: Production image with only runtime dependencies

```bash
# Direct Docker commands
docker compose build
docker compose up -d
docker compose logs -f
```

## Architecture

### Single-File MCP Server

The entire MCP server implementation is in [frontapp_mcp.ts](frontapp_mcp.ts). It follows this structure:

1. **FrontappMCPServer class** - Main server implementation
   - `constructor()` - Initializes MCP Server and Axios instance with API token
   - `setupHandlers()` - Registers tool, resource, and request handlers
   - Private methods for each API operation (list/get/create/update/delete)

2. **MCP Protocol Implementation**:
   - `ListToolsRequestSchema` - Returns 20+ tool definitions
   - `CallToolRequestSchema` - Routes tool calls to appropriate API methods
   - `ListResourcesRequestSchema` - Exposes quick-access resources
   - `ReadResourceRequestSchema` - Fetches resource data

3. **Communication Flow**:
   ```
   Claude Code
     ↓ (stdio pipe)
   docker-mcp-wrapper.sh
     ↓ (docker exec)
   Docker Container
     ↓ (node dist/index.js)
   FrontappMCPServer
     ↓ (HTTPS with Bearer token)
   Frontapp API (api2.frontapp.com)
   ```

### Tool Handler Pattern

All tool calls are routed through a single switch-case handler in `CallToolRequestSchema`:

```typescript
switch (name) {
  case 'list_conversations':
    result = await this.listConversations(args);
    break;
  case 'get_conversation':
    result = await this.getConversation(args.conversation_id);
    break;
  // ... 20+ tools total
}
```

Each private method corresponds to one Frontapp API endpoint and handles parameter transformation.

### Resource Endpoints

Four pre-configured resources provide instant access to common data:
- `frontapp://conversations/recent` - Last 20 conversations
- `frontapp://teammates` - All teammates
- `frontapp://inboxes` - All inboxes
- `frontapp://tags` - All tags

These are useful for Claude to quickly understand the Frontapp workspace without explicit queries.

### Error Handling

The server catches Axios errors and extracts meaningful messages:
```typescript
catch (error: any) {
  const errorMessage = error.response?.data?.message || error.message;
  return {
    content: [{ type: 'text', text: `Error: ${errorMessage}` }],
    isError: true,
  };
}
```

## API Integration

### Authentication

The server uses Bearer token authentication via environment variable:
```typescript
this.axiosInstance = axios.create({
  baseURL: 'https://api2.frontapp.com',
  headers: {
    'Authorization': `Bearer ${apiToken}`,
    'Content-Type': 'application/json',
  },
});
```

### Conversation Search Syntax

Front supports rich query syntax for `search_conversations`:
- `status:open` / `status:archived` - Filter by status
- `tag:urgent` - Filter by tag
- `assignee:me` / `is:unassigned` - Filter by assignment
- `inbox:support` - Filter by inbox
- `after:2024-01-01` / `before:2024-12-31` - Date ranges
- Combine with `AND`, `OR`: `status:open AND tag:urgent`

### Pagination

Most list endpoints support pagination:
- `limit` - Number of results (max 100, default 50)
- `page_token` - Token from previous response's pagination.next

### Message Types

When replying to conversations, there are two types:
- `reply` - External reply to conversation participants (requires `channel_id`)
- `comment` - Internal comment only visible to teammates

## Docker Deployment

### Wrapper Script Architecture

Claude Code doesn't directly execute Docker commands. Instead:

1. **Claude Code config** points to `docker-mcp-wrapper.sh`
2. **Wrapper script**:
   - Ensures container is running (`docker compose up -d`)
   - Pipes stdio to/from container (`docker exec -i`)
   - Handles cleanup on exit

This allows the containerized server to appear as a native stdio process to Claude Code.

### Security

The Docker setup includes:
- Non-root user (uid 1001) for container execution
- Multi-stage build to minimize attack surface
- Alpine Linux base for minimal footprint
- No unnecessary packages in production image
- `.env` file for API token (never commit this!)

### Configuration Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zqushair/Frontapp-MCP](https://github.com/zqushair/Frontapp-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
