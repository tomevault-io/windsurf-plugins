---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WeChat Official Account MCP (Model Context Protocol) server that provides AI applications like Claude Desktop, Cursor, and Trae AI with seamless integration to WeChat Official Account APIs. The project implements MCP tools for managing WeChat media, drafts, and publishing functionality.

**Tech Stack**: Node.js 18+, TypeScript, MCP SDK, SQLite, Axios, Zod, Express (for SSE transport), crypto-js (for encryption)

## Essential Commands

### Development

```bash
# Build the project (cleans and compiles TypeScript)
npm run build

# Production build (with stricter tsconfig)
npm run build:prod

# Type checking without emitting files
npm run check

# Lint code
npm run lint

# Run tests
npm test
```

### Running the Server

```bash
# Development mode (direct with tsx)
npm run dev -- mcp -a <app_id> -s <app_secret>

# Start built server
node dist/src/cli.js mcp -a <app_id> -s <app_secret>

# Using npx (recommended for users)
npx wechat-official-account-mcp mcp -a <app_id> -s <app_secret>
```

### CLI Options

- `-a, --app-id <appId>`: WeChat App ID (required)
- `-s, --app-secret <appSecret>`: WeChat App Secret (required)
- `-m, --mode <mode>`: Transport mode - `stdio` (default) or `sse`
- `-p, --port <port>`: Port for SSE mode (default: 3000)

### Packaging & Release

```bash
# Local package testing
npm run pack:test

# Dry run pack (show what would be included)
npm run pack:dry

# Full build with verification
./scripts/build.sh
```

## Architecture

### Core Components

**Entry Points**:
- `src/cli.ts` - CLI entry point using Commander.js
- `src/index.ts` - Module exports for library usage

**MCP Server Layer** (`src/mcp-server/`):
- `shared/init.ts` - Server initialization logic (`initWechatMcpServer`, `initMcpServerWithTransport`)
- `shared/types.ts` - Server configuration types (`McpServerOptions`)
- `transport/stdio.ts` - stdio transport for local MCP clients (default for Claude Desktop)
- `transport/sse.ts` - SSE (Server-Sent Events) transport for remote/web clients using Express

**Tool Layer** (`src/mcp-tool/`):
- `index.ts` - `WechatMcpTool` class that manages all MCP tools (tool registry, execution, error handling)
- `types.ts` - Core type definitions (WechatConfig, AccessTokenInfo, MediaInfo, McpTool, etc.)
- `tools/` - Individual tool implementations (auth-tool, media-upload-tool, draft-tool, etc.)

**Service Layer**:
- `src/auth/auth-manager.ts` - Manages WeChat credentials and Access Token lifecycle (with auto-refresh)
- `src/wechat/api-client.ts` - Axios-based HTTP client with automatic token injection via request interceptor
- `src/storage/storage-manager.ts` - SQLite persistence with optional AES-256 encryption

**Utilities** (`src/utils/`):
- `logger.ts` - Logging utility with debug/info/error levels
- `validation.ts` - Zod schemas and sanitization functions for input validation
- `db-init.ts` - Database initialization and schema management

### Data Flow

1. CLI args → `McpServerOptions` → `initMcpServerWithTransport()`
2. Server creates `AuthManager` (handles credentials/token) and `WechatMcpTool` (tool registry)
3. Tools are registered to MCP server via `WechatMcpTool.registerTools()`
4. When a tool is called, handler receives `params` and `WechatApiClient`, returns `WechatToolResult`

### Tool Implementation Pattern

Each tool in `src/mcp-tool/tools/` follows this pattern:
```typescript
// 1. Define Zod schema for validation (reuse schemas from utils/validation.ts when possible)
const schema = z.object({
  action: z.enum(['action1', 'action2']),
  param: z.string().optional(),
});

// 2. Export McpTool with handler
export const toolName: McpTool = {
  name: 'tool_name',
  description: 'Tool description',
  inputSchema: { /* ZodRawShape - directly passed to registerTool */ },
  handler: async (params: unknown, apiClient: WechatApiClient) => {
    const validated = schema.parse(params);
    // Execute logic using apiClient methods
    return { content: [{ type: 'text', text: 'result' }] };
  }
};
```

**Key Points**:
- All tools are exported from `src/mcp-tool/tools/index.ts` as `mcpTools` array
- Tool handlers receive `(params, apiClient)` - `params` is the raw input, validate it with Zod
- Return format must match `WechatToolResult` with `content` array containing text/image/resource items
- The `WechatMcpTool.registerTools()` wraps handlers with error handling, returning formatted errors
- Use validation schemas from `utils/validation.ts` (e.g., `mediaIdSchema`, `appIdSchema`, `articleTitleSchema`)
- For file operations, use `isValidMediaType()` and `isValidFileSize()` from validation utilities

### Security Features

- **Encryption**: Set `WECHAT_MCP_SECRET_KEY` environment variable to enable AES-256 encryption for sensitive fields (app_secret, token, encoding_aes_key, access_token). Encrypted values are stored with `enc:` prefix in the database.
- **CORS**: Configure `CORS_ORIGIN` as comma-separated whitelist for SSE mode (e.g., `https://domain1.com,https://domain2.com`). **Never use `*` in production**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xwang152-jack/wechat-official-account-mcp](https://github.com/xwang152-jack/wechat-official-account-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
