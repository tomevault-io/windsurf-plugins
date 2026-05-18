---
trigger: always_on
description: This document provides context and guidelines for GitHub Copilot when working with the MCP Server Trello codebase.
---

# GitHub Copilot Instructions for MCP Server Trello

This document provides context and guidelines for GitHub Copilot when working with the MCP Server Trello codebase.

## Project Overview

MCP Server Trello is a Model Context Protocol (MCP) server that provides tools for interacting with Trello boards. The project is powered by Bun runtime for maximum performance and is written in TypeScript with strict type safety.

## Tech Stack

- **Runtime**: Bun (v1.0.0+), but also compatible with Node.js via npm/npx
- **Language**: TypeScript 5.3+
- **Build Tool**: TypeScript Compiler (tsc)
- **Primary Dependencies**:
  - `@modelcontextprotocol/sdk` - MCP protocol implementation
  - `axios` - HTTP client for Trello API
  - `zod` - Runtime type validation
  - `form-data` - File upload handling
- **Code Quality**: ESLint + Prettier
- **Testing**: Not yet implemented (contributions welcome)

## Architecture

### Project Structure

```
src/
├── index.ts                 # Main MCP server implementation and tool registration
├── trello-client.ts         # Trello API client with rate limiting
├── rate-limiter.ts          # Token bucket rate limiter implementation
├── types.ts                 # TypeScript type definitions
├── validators.ts            # Zod schemas for input validation
├── health/                  # Health monitoring endpoints
│   ├── health-endpoints.ts  # Health check MCP tools
│   └── health-monitor.ts    # Health monitoring logic
└── evals/                   # Evaluation tests (using mcp-evals)
```

### Key Components

1. **TrelloServer** (`src/index.ts`): Main server class that:
   - Registers MCP tools via `registerTool()`
   - Handles tool invocations
   - Manages error handling
   - Sets up health monitoring

2. **TrelloClient** (`src/trello-client.ts`): API client that:
   - Manages authentication (API key + token)
   - Implements rate limiting (300 req/10s per key, 100 req/10s per token)
   - Provides methods for all Trello operations
   - Handles board/workspace persistence via `~/.trello-mcp/config.json`

3. **Rate Limiter** (`src/rate-limiter.ts`):
   - Token bucket algorithm implementation
   - Separate buckets for API key and token limits
   - Automatic request queuing when limits are reached

## Development Guidelines

### Building

```bash
# Preferred (if Bun is installed)
bun run build

# Alternative (works anywhere)
npx tsc
```

The build output goes to the `build/` directory.

### Code Style

- **Indentation**: 2 spaces (no tabs)
- **Quotes**: Single quotes preferred
- **Semicolons**: Required
- **Line Length**: 100 characters max
- **Import Style**: ES modules only
- **Trailing Commas**: ES5 style
- **Arrow Functions**: Avoid parentheses for single parameters

Run linting:
```bash
npx eslint src --ext .ts
```

Run formatting:
```bash
npx prettier --write src
```

### TypeScript Guidelines

- **Strict mode enabled**: All code must comply with strict TypeScript
- **No implicit any**: Always provide explicit types
- **Return types**: Explicit return types are preferred for public methods
- **Error handling**: Use `try/catch` and return structured error objects
- **Async/await**: Preferred over raw promises

### Environment Variables

Required:
- `TRELLO_API_KEY` - Get from https://trello.com/app-key
- `TRELLO_TOKEN` - Generate using API key authorization flow

Optional:
- `TRELLO_BOARD_ID` - Default board (can be changed via `set_active_board` tool)
- `TRELLO_WORKSPACE_ID` - Initial workspace (can be changed via `set_active_workspace` tool)

### Adding New Tools

When adding a new MCP tool:

1. **Define the tool schema** using Zod in the `registerTool()` call
2. **Implement validation** using the validators in `src/validators.ts`
3. **Add the method** to `TrelloClient` if it requires API calls
4. **Handle errors gracefully** using the `handleError()` pattern
5. **Update README.md** with the new tool documentation
6. **Consider rate limiting** - all API calls are automatically rate-limited

Example tool structure:
```typescript
this.server.registerTool(
  'tool_name',
  {
    title: 'Human Readable Title',
    description: 'Clear description of what this tool does',
    inputSchema: {
      param1: z.string().describe('Description of param1'),
      param2: z.string().optional().describe('Optional description'),
    },
  },
  async ({ param1, param2 }) => {
    try {
      const result = await this.trelloClient.someMethod(param1, param2);
      return {
        content: [{ type: 'text' as const, text: JSON.stringify(result, null, 2) }],
      };
    } catch (error) {
      return this.handleError(error);
    }
  }
);
```

### Working with the Trello API

- **API Documentation**: https://developer.atlassian.com/cloud/trello/rest/
- **Rate Limits**: Automatically handled by `RateLimiter` class
- **Authentication**: API key + token passed via query parameters
- **Error Handling**: Use `McpError` from the SDK for user-facing errors
- **Board Management**: Support both explicit `boardId` parameter and default board

### Common Patterns

**Date Formats**:
- `dueDate`: Full ISO 8601 with time (e.g., `2023-12-31T12:00:00Z`)
- `start`: Date only in YYYY-MM-DD format (e.g., `2025-08-05`)

**Board ID Resolution**:
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [delorenj/mcp-server-trello](https://github.com/delorenj/mcp-server-trello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
