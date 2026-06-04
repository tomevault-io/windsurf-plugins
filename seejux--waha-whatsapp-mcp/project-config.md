---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WAHA MCP Server is a Model Context Protocol server that enables AI assistants to interact with WhatsApp through the WAHA (WhatsApp HTTP API) platform. It provides tools for chat management, message operations, and real-time webhook notifications.

**Tech Stack**: TypeScript, Node.js, MCP SDK, WAHA API, Express (webhooks), ngrok (webhook tunneling)

## Build & Development Commands

```bash
# Build the project
npm run build

# Development mode with auto-reload
npm run dev

# Production mode (stdio transport for local MCP clients)
npm start

# Test with MCP Inspector (web-based testing tool)
npm run inspector
```

## Configuration

Required environment variables in `.env`:
- `WAHA_BASE_URL`: WAHA server URL (e.g., `http://localhost:3000`)
- `WAHA_API_KEY`: API key for WAHA authentication
- `WAHA_SESSION`: WhatsApp session name (default: `default`)

Optional webhook configuration:
- `WEBHOOK_ENABLED`: Enable webhook support (default: `false`)
- `WEBHOOK_PORT`: Local webhook server port (default: `3001`)
- `WEBHOOK_HMAC_KEY`: Secret key for webhook signature validation
- `NGROK_AUTHTOKEN`: ngrok auth token for public tunnel
- `WEBHOOK_AUTO_START`: Auto-start webhooks on server startup (default: `true`)

## Architecture

### Three-Layer Architecture

1. **MCP Server Layer** (`src/index.ts`)
   - `WAHAMCPServer` class orchestrates all functionality
   - Handles MCP protocol (tools, resources, stdio transport)
   - Manages lifecycle (startup, cleanup, SIGINT handling)
   - Conditionally loads webhook system via dynamic import

2. **WAHA Client Layer** (`src/client/waha-client.ts`)
   - `WAHAClient` class handles all HTTP communication with WAHA API
   - Uses native `fetch` with `X-Api-Key` header authentication
   - Custom `WAHAError` class for API error handling
   - Methods: `getChatsOverview()`, `getChatMessages()`, `sendTextMessage()`, `markChatAsRead()`, `updateSessionWebhook()`

3. **Domain Logic**
   - **Tools** (`src/tools/`): MCP tool implementations with formatters
   - **Resources** (`src/resources/`): MCP resources with caching (LRU, 5 min TTL)
   - **Webhooks** (`src/webhooks/`): Real-time event handling system

### Resource System

Resources are implemented using an abstract base class pattern:

- **BaseResource** (`src/resources/base/BaseResource.ts`): Abstract class with helper methods for URI parsing, parameter validation
- **Implementations** (`src/resources/implementations/`): Concrete resources (ChatsOverviewResource, ChatMessagesResource)
- **ResourceManager** (`src/resources/manager/ResourceManager.ts`): Registry pattern for resource discovery and routing
- **ResourceCache** (`src/resources/cache/ResourceCache.ts`): LRU cache with automatic pruning and TTL

Resources support URI parameters (e.g., `waha://chats/overview?limit=10&offset=0`).

### Webhook System

The webhook system provides real-time WhatsApp event notifications:

- **WebhookServer** (`src/webhooks/server/WebhookServer.ts`): Express HTTP server on configurable port
  - HMAC signature validation for security
  - Event dispatcher to registered handlers
  - Health check endpoint at `/health`
- **Event Handlers** (`src/webhooks/handlers/`):
  - `BaseEventHandler`: Abstract base with `handleEvent()` and `sendNotification()`
  - `MessageHandler`: Processes `message` events (incoming messages)
  - `AckHandler`: Processes `message.ack` events (delivery/read receipts)
  - `StateHandler`: Processes `state.change` events (connection status)
- **NgrokManager** (`src/webhooks/ngrok/NgrokManager.ts`): Manages ngrok tunnel for public webhook URL
- **WebhookManager** (`src/webhooks/index.ts`): Coordinates server, handlers, ngrok, and WAHA webhook configuration

Webhooks are only loaded when `WEBHOOK_ENABLED=true` to avoid loading ngrok dependencies unnecessarily.

## Important Implementation Details

### Chat ID Format
- Individual chats: `<phone_number>@c.us` (e.g., `1234567890@c.us`)
- Group chats: `<group_id>@g.us` (e.g., `123456789012345678@g.us`)
- Validated using `WAHAClient.validateChatId()`

### Error Handling
- All WAHA API calls are wrapped in try-catch
- `WAHAError` provides structured error responses with status codes
- Tool handlers return `{ isError: true }` on failure
- Webhook handlers catch and log errors without crashing

### TypeScript Configuration
- Module system: ES2022 with Node16 module resolution
- All imports must use `.js` extension (e.g., `import { x } from './file.js'`)
- Strict mode enabled
- Output to `dist/` directory

### MCP Protocol
- Server capabilities: `tools` and `resources`
- Transport: stdio (for local clients like Claude Desktop)
- Tool calls handled via `CallToolRequestSchema`
- Resources handled via `ListResourcesRequestSchema` and `ReadResourceRequestSchema`

## Adding New Features

### Adding a New Tool
1. Add tool schema to `ListToolsRequestSchema` handler in `src/index.ts`
2. Add case to switch statement in `CallToolRequestSchema` handler
3. Implement handler method (e.g., `handleNewTool()`)
4. Add corresponding method to `WAHAClient` if API call needed
5. Create formatter function in `src/tools/formatters.ts`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seejux/waha-whatsapp-mcp](https://github.com/seejux/waha-whatsapp-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
