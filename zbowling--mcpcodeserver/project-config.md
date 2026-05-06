---
trigger: always_on
description: This document provides comprehensive guidance for Claude Code (claude.ai/code) and other AI agents when working with this codebase. It covers project structure, implementation details, development workflows, and testing strategies.
---

# AGENTS.md - mcpcodeserver Project Structure

This document provides comprehensive guidance for Claude Code (claude.ai/code) and other AI agents when working with this codebase. It covers project structure, implementation details, development workflows, and testing strategies.

## Project Overview

**mcpcodeserver** is an MCP (Model Context Protocol) proxy server that transforms tool calling into code generation. It connects to child MCP servers as a client, discovers their tools, and exposes two meta-tools that allow LLMs to generate and execute TypeScript code that calls those tools.

### Key Innovation

Instead of the traditional pattern:

```
LLM → Tool Call 1 → LLM → Tool Call 2 → LLM → Tool Call 3 → LLM
```

This enables:

```
LLM → Generate Code → Execute Code (calls Tool 1, 2, 3 internally) → Result
```

## Directory Structure

```
mcpcodeserver/
├── src/
│   ├── index.ts           # CLI entry point and configuration loading
│   ├── server.ts          # Main MCP server implementation
│   ├── client-manager.ts  # Manages connections to child MCP servers
│   ├── codegen.ts         # TypeScript code generation from tool schemas
│   ├── sandbox.ts         # VM-based code execution sandbox
│   └── types.ts           # TypeScript type definitions
├── tests/
│   ├── unit/              # Unit tests
│   ├── integration/       # Integration tests
│   ├── vm-tests/          # TypeScript test programs for VM execution
│   └── mock-server/       # Pizza Shop test server (Python)
├── dist/                  # Compiled JavaScript output (generated)
├── node_modules/          # Dependencies (generated)
├── package.json           # Project metadata and dependencies
├── tsconfig.json          # TypeScript compiler configuration
├── mcp.json.example       # Example configuration file
├── README.md              # User documentation
├── AGENTS.md              # This file - developer/agent documentation
└── .gitignore             # Git ignore rules
```

## Using with MCP Clients

**Recommended:** Use `npx` to run mcpcodeserver without installation:

```json
{
  "mcpServers": {
    "codeserver": {
      "command": "npx",
      "args": ["-y", "mcpcodeserver", "--config", "/path/to/mcp.json"]
    }
  }
}
```

**From GitHub directly:**
```json
{
  "mcpServers": {
    "codeserver": {
      "command": "npx",
      "args": ["-y", "github:zbowling/mcpcodeserver", "--config", "/path/to/mcp.json"]
    }
  }
}
```

**Other package managers:** `yarn dlx`, `pnpm dlx`, `bunx` all work similarly.

See [examples/](./examples/) for more configuration options.

## Child Server Configuration Format (mcp.json)

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
      "env": { "DEBUG": "false" }
    },
    "api-server": {
      "url": "http://localhost:3000/mcp",
      "transport": "sse"
    }
  }
}
```

**Transport types:**
- **stdio**: Spawns child process (requires `command`, optional `args` and `env`)
- **sse**: HTTP Server-Sent Events (requires `url`, `transport: "sse"`)

## Module Documentation

### 1. index.ts - CLI Entry Point

**Purpose:** Command-line interface and application bootstrapping

**Key Functions:**

- `parseArgs()` - Parse command-line arguments (--config, --help)
- `showHelpMessage()` - Display usage information
- `loadConfig(configPath)` - Load and validate mcp.json configuration
- `main()` - Main entry point that orchestrates startup

**Flow:**

1. Parse command-line arguments
2. Load configuration from mcp.json
3. Validate configuration structure
4. Start the MCP server via `runServer()`

**Configuration Format:**

```typescript
interface MCPConfig {
  mcpServers: Record<string, MCPServerConfig>
}

interface MCPServerConfig {
  command?: string;      // For stdio transport
  args?: string[];       // Command arguments
  env?: Record<string, string>;  // Environment variables
  url?: string;          // For HTTP/SSE transport
  transport?: "stdio" | "sse";   // Transport type
}
```

### Tool Naming Convention

- **Discovery:** Child tools are cached as `serverName.toolName` (e.g., `filesystem.read_file`)
- **Generated functions:** Converted to `serverName_toolName` (e.g., `filesystem_read_file()`)
- This prevents naming conflicts when multiple servers expose similar tools

### Data Flow: execute_toolcall_script

1. User calls `execute_toolcall_script({ code: "...", timeout: 30000 })`
2. Server validates code and timeout parameters
3. `executeSandbox()` creates VM context with injected `__callTool()` function
4. `generateRuntimeStubs()` creates tool function wrappers
5. Code executed: `(async () => { stubs + userCode })()`
6. User code calls `await filesystem_read_file({ path: "..." })`
7. Stub calls `__callTool("filesystem.read_file", params)`
8. `clientManager.callTool()` routes to appropriate child server
9. Result returned to user code, which continues execution
10. Final result, console logs, and errors formatted and returned

### 2. server.ts - MCP Server Implementation

**Purpose:** Main MCP server that exposes tools to parent clients


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zbowling/mcpcodeserver](https://github.com/zbowling/mcpcodeserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
