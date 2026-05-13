---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Reloaderoo** is a dual-purpose MCP (Model Context Protocol) tool that serves as both:

1. **MCP Server (Proxy Mode)** - A transparent development proxy for hot-reloading MCP servers
2. **MCP Client/CLI Tool (Inspection Mode)** - A debugging interface for inspecting MCP servers

## 🔄 Proxy Mode (Default Mode)

**Primary Purpose:** Hot-reloading MCP server development with transparent forwarding

### Architecture: Proxy Mode

```
MCP Client ↔ Reloaderoo Proxy (MCP Server) ↔ Child MCP Server
(e.g., Claude)   (transparent proxy + restart_server tool)   (your-server)
```

#### Key Characteristics:
- **IS an MCP Server** - Reloaderoo itself acts as an MCP server
- **Transparent Forwarding** - All MCP messages pass through seamlessly
- **Capability Augmentation** - Adds `restart_server` tool to child server's capabilities
- **Session Persistence** - Client connection remains active during server restarts
- **Default Mode** - Runs when using `reloaderoo` or `reloaderoo proxy`

#### Core Components (Proxy Mode):
- `MCPProxy` - Main proxy implementation
- `ProcessManager` - Child server lifecycle management
- `MessageRouter` - JSON-RPC message forwarding
- `CapabilityAugmenter` - Modifies `InitializeResult` to add proxy capabilities
- `RestartHandler` - Implements the `restart_server` tool

#### Message Flow (Proxy Mode):
1. Client sends MCP request → Reloaderoo Proxy
2. **If `restart_server` tool call:** Handle internally, restart child, notify client
3. **All other requests:** Forward to child server → return response to client
4. **Initialize handshake:** Intercept and add `restart_server` to capabilities
5. **Notifications:** Send `tools/list_changed` after restarts

#### Usage (Proxy Mode):
```bash
# Basic proxy usage (default mode)
reloaderoo -- node /path/to/my-mcp-server.js

# Explicit proxy mode with options  
reloaderoo proxy --log-level debug --max-restarts 5 -- node server.js
```

#### Configuration for MCP Clients (Proxy Mode):
```json
{
  "mcpServers": {
    "myDevelopmentServer": {
      "command": "node",
      "args": [
        "/path/to/reloaderoo/dist/bin/reloaderoo.js", 
        "proxy",
        "--log-level", "debug",
        "--",
        "node", 
        "/path/to/my-server.js"
      ]
    }
  }
}
```

## 🔍 Inspection Mode

**Primary Purpose:** MCP server debugging and development testing

### Dual Interface: CLI + MCP Server

Inspection mode serves **two different use cases**:

#### 1. CLI Interface (Primary Use Case)
**For AI agents that DON'T need direct MCP access** - Acts as an MCP client via CLI

```bash
# AI agents can use these commands directly without MCP configuration
reloaderoo inspect list-tools -- node my-server.js
reloaderoo inspect call-tool echo --params '{"message":"test"}' -- node my-server.js
reloaderoo inspect server-info -- node my-server.js
```

**Benefits for AI Agents:**
- ✅ No MCP server configuration required in AI session
- ✅ Direct CLI access to any MCP server for testing
- ✅ Perfect for development debugging workflows
- ✅ JSON formatted responses for easy parsing

#### 2. MCP Server Interface (Secondary Use Case)  
**For MCP clients that need protocol access** - Reloaderoo as inspection MCP server

```bash
# Start inspection MCP server
reloaderoo inspect mcp -- node /path/to/my-server.js
```

### Architecture: Inspection Mode

#### CLI Interface Architecture:
```
AI Agent ↔ CLI Commands ↔ Reloaderoo Inspector ↔ Child MCP Server
         (direct CLI calls)  (acts as MCP client)   (your-server)
```

#### MCP Server Interface Architecture:
```
MCP Client ↔ Reloaderoo Inspector (MCP Server) ↔ Child MCP Server  
(e.g., Claude)  (8 debug tools only)              (not directly exposed)
```

#### Key Characteristics:
- **CLI-First Design** - Primarily designed for command-line debugging
- **MCP Client Functionality** - Reloaderoo acts as a client to your MCP server
- **Debug Tools Only** - Exposes 8 inspection tools, child server tools accessed via `call_tool`
- **Development Focus** - Specifically for MCP server development and debugging

### Complete Inspection Tools List

#### Available Tools (8 total):

1. **`list_tools`** - List all available tools from child server
   ```bash
   reloaderoo inspect list-tools -- node server.js
   ```

2. **`call_tool`** - Call any tool on the child server
   ```bash
   reloaderoo inspect call-tool <tool-name> --params '{"param":"value"}' -- node server.js
   ```

3. **`list_resources`** - List all available resources from child server
   ```bash
   reloaderoo inspect list-resources -- node server.js
   ```

4. **`read_resource`** - Read a specific resource from child server
   ```bash
   reloaderoo inspect read-resource <resource-uri> -- node server.js
   ```

5. **`list_prompts`** - List all available prompts from child server
   ```bash
   reloaderoo inspect list-prompts -- node server.js
   ```

6. **`get_prompt`** - Get a specific prompt from child server
   ```bash
   reloaderoo inspect get-prompt <prompt-name> -- node server.js
   ```

7. **`get_server_info`** - Get comprehensive server information and capabilities
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cameroncooke/reloaderoo](https://github.com/cameroncooke/reloaderoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
