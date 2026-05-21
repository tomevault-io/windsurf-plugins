---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides AI agents with database operations, serverless code deployment, and file management capabilities on the Codehooks.io platform. The server wraps the Codehooks CLI (`coho`) and exposes its functionality through MCP tools.

## Build and Development Commands

### Build
```bash
npm run build
```
Compiles TypeScript to JavaScript, copies the ChatGPT prompt markdown file to build directory, and sets execute permissions on the output.

### Watch Mode
```bash
npm run watch
```
Compiles TypeScript in watch mode for development.

### MCP Inspector (Testing)
```bash
npm run inspector
```
Runs the MCP inspector for testing the server locally using the configuration in `mcp-dev.json`.

### Docker Build
```bash
docker build -t codehooks-mcp .
```

## Architecture

### Single File Design
The entire MCP server is implemented in a single file (`src/index.ts`) with approximately 1500 lines of code. This monolithic design keeps all tool definitions, schemas, and handlers in one place.

### Tool Implementation Pattern
Each MCP tool follows this pattern:
1. **Schema Definition**: Zod schema for input validation (e.g., `queryCollectionSchema`)
2. **Type Inference**: TypeScript type derived from schema (e.g., `type QueryCollectionArgs = z.infer<typeof queryCollectionSchema>`)
3. **Tool Descriptor**: Object with `name`, `description`, `schema`, and `inputSchema` in the `tools` array
4. **Handler Logic**: Switch case in the `CallToolRequestSchema` handler that constructs CLI arguments and calls `executeCohoCommand()`

### CLI Command Wrapper
The core function `executeCohoCommand(args: string[])` (src/index.ts:529):
- Executes the Codehooks CLI (`coho`) with provided arguments
- Automatically injects the admin token from environment variables
- Sanitizes all output and error messages to prevent token exposure (replaces admin token with `***`)
- Throws `McpError` on command failure with sanitized error messages

### Configuration
Three environment variables configure the server (src/index.ts:26-36):
- `CODEHOOKS_PROJECT_NAME`: Project identifier
- `CODEHOOKS_ADMIN_TOKEN`: Authentication token (must be kept secret)
- `CODEHOOKS_SPACE`: Deployment space (defaults to "dev")

### Security: Token Sanitization
All CLI output, error messages, stderr, and stdout are sanitized to replace the admin token with `***`. This prevents accidental exposure in logs or error messages. The sanitization happens in:
- `executeCohoCommand()` function for successful command output
- Error handling with comprehensive sanitization of all error properties (message, cmd, stdout, stderr)

### File Operations with Temporary Storage
Tools that handle file content (deploy, import, add_schema, file_upload) follow this pattern:
1. Create temporary file in `/tmp/` with unique name
2. Write content to temporary file
3. Execute CLI command with temporary file path
4. Clean up temporary file (with try/catch to ensure cleanup on error)

Example: `deploy_code` creates a temporary directory, writes all files including auto-generated `package.json`, runs `npm install`, then executes deployment.

### Documentation Integration
The server includes a `docs` tool that combines:
- MCP server overview and configuration
- The complete ChatGPT prompt loaded from `src/chatgpt-prompt.md`
- API URL format documentation
- Links to external resources

The ChatGPT prompt is loaded at server startup via `loadChatGPTPrompt()` (src/index.ts:41-48) and stored in memory.

## Key Design Patterns

### Default Limit Pattern
The `query_collection` tool applies a default limit of 100 records unless `count` is true. This prevents accidentally returning massive datasets.

### Auto-Generated package.json
The `deploy_code` tool automatically generates a minimal `package.json` if one isn't provided, ensuring deployments always have proper module configuration with `"type": "module"` and the `codehooks-js` dependency.

### Content vs File Path
Several tools (import, export, file_upload) accept either file content as a string or a file path, providing flexibility for different use cases:
- `content` parameter: Data passed directly (creates temp file)
- `filepath` parameter: Reference to existing file in Docker container

### Working Directory Management for Deployment
The `deploy_code` handler temporarily changes the process working directory to the temporary deployment directory, executes the deployment, then restores the original working directory (src/index.ts:781-827). This is necessary because the Codehooks CLI expects to run from the project root.

## Important Implementation Details

### Error Handling Hierarchy
1. `McpError` exceptions are thrown and propagated directly
2. `ZodError` validation errors return structured error response with `isError: true`
3. Generic errors are caught and wrapped in error response format
4. All CLI command errors are sanitized before being thrown as `McpError`

### Output Format Detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RestDB/codehooks-mcp-server](https://github.com/RestDB/codehooks-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
