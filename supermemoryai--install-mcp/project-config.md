---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and similar coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) and similar coding agents when working with code in this repository.

## Development Commands

- `bun run build` - Build the project using tsup
- `bun run build:watch` - Build with watch mode
- `bun run start` - Run the CLI locally using ts-node
- `bun run start:node` - Run the built CLI from dist/
- `bun run test` - Run bun test suite
- `bun run test:watch` - Run tests in watch mode
- `bun run lint` - Run Biome linter
- `bun run lint:fix` - Run Biome linter with auto-fix
- `bun run format` - Check code formatting with Biome
- `bun run format:fix` - Auto-format code with Biome
- `bun run compile` - Type-check with TypeScript compiler

## Architecture Overview

This is a CLI tool for installing and managing MCP (Model Context Protocol) servers across different AI clients. The tool supports 14+ AI clients with different configuration formats and file locations.

### Core Entry Points

- `bin/run.ts` - Main CLI entry point using yargs for command parsing
- `bin/run` - Shell script that requires the built version

### Core Components

- `src/commands/install.ts` - Main install command logic handling URL and command-based installations
- `src/client-config.ts` - Client configuration management with platform-specific paths
- `src/detect-transport.ts` - Transport detection logic (stdio, sse, etc.)
- `src/logger.ts` - Logging utilities using consola
- `src/index.ts` - Command registration and CLI setup

## Client Support System

### Currently Supported Clients (14 total)

**Desktop Applications:**

- Claude Desktop (`claude-desktop`)
- Cursor (`cursor`)
- Windsurf (`windsurf`)
- Witsy (`witsy`)
- Enconvo (`enconvo`)

**VS Code Extensions:**

- Cline (`cline`)
- Roo-Cline (`roo-cline`)
- VS Code native (`vscode`)

**CLI Tools:**

- Warp (`warp`) - Outputs config for manual copy-paste
- Gemini CLI (`gemini-cli`)
- Claude Code (`claude-code`)
- Goose (`goose`)
- Zed (`zed`)
- Codex (`codex`)

### Configuration Formats Supported

- **JSON** (default): Most clients
- **YAML**: Goose client
- **TOML**: Codex client

## Installation Patterns

### Three Installation Types

1. **Simple Package Names**: `mcp-package-name` → converts to `npx mcp-package-name`
2. **Full Commands**: `'node server.js --port 3000'` → preserved as-is
3. **Remote URLs**: `https://api.example.com/server` → uses mcp-remote approach

### Server Configuration Structures

Different clients require different config structures:

**Standard Format (Claude, Cline, etc.):**

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["package-name"]
    }
  }
}
```

**Goose Format (YAML):**

```yaml
extensions:
  server-name:
    name: server-name
    cmd: npx
    args: [package-name]
    enabled: true
    envs: {}
    type: stdio
    timeout: 300
```

**Zed Format:**

```json
{
  "lsp": {
    "server-name": {
      "source": "custom",
      "command": "npx",
      "args": ["package-name"],
      "env": {}
    }
  }
}
```

## Adding New AI Clients

### Step 1: Define Client Configuration

Add your client to the `getClientPaths()` function in `src/client-config.ts`:

```typescript
export function getClientPaths(): Record<string, ClientFileTarget> {
  const { homeDir, platform, baseDir } = getPlatformPaths();

  return {
    // ... existing clients ...

    "new-client": {
      type: "file",
      path: path.join(baseDir, "NewClient", "config.json"),
      configKey: "mcpServers", // or 'mcp.servers', 'lsp', etc.
      format: "json", // optional: 'yaml' or 'toml' if not JSON
      localPath: path.join(process.cwd(), ".new-client", "config.json"), // optional local config
    },
  };
}
```

**Configuration Options:**

- `type: 'file'` - File-based configuration (only supported type currently)
- `path` - Absolute path to global config file
- `localPath` - Optional path for project-local config (used with `--local` flag)
- `configKey` - Nested key path (e.g., 'mcpServers', 'mcp.servers', 'lsp')
- `format` - File format: 'json' (default), 'yaml', or 'toml'

### Step 2: Add to Supported Clients

Add your client name to the `clientNames` array in `src/client-config.ts`:

```typescript
export const clientNames = [
  "claude-desktop",
  "cline",
  "roo-cline",
  "windsurf",
  "witsy",
  "enconvo",
  "cursor",
  "warp",
  "gemini-cli",
  "vscode",
  "claude-code",
  "goose",
  "zed",
  "codex",
  "new-client", // Add your client here
];
```

### Step 3: Handle Custom Server Structure (Optional)

If your client requires a different server configuration structure, add logic to `setServerConfig()` in `src/commands/install.ts`:

```typescript
function setServerConfig(
  client: string,
  servers: Record<string, any>,
  serverName: string,
  serverConfig: Record<string, any>,
) {
  switch (client) {
    case "new-client":
      // Custom structure for your client
      servers[serverName] = {
        name: serverName,
        executable: serverConfig.command,
        arguments: serverConfig.args,
        enabled: true,
        timeout: 300,
      };
      break;

    // ... existing cases ...
  }
}
```

### Step 4: Add Tests

Add test cases in `src/client-config.test.ts`:

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supermemoryai/install-mcp](https://github.com/supermemoryai/install-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
