---
trigger: always_on
description: **Developer & Contributor Guide for Claude Code**
---

# CLAUDE.md

**Developer & Contributor Guide for Claude Code**

This file provides guidance to Claude Code (claude.ai/code) and human contributors when working with code in this repository.

## Project Overview

MIDICtrl is an HTTP-based MCP (Model Context Protocol) server that enables LLMs to control and interact with MIDI devices. Built with Elixir, it uses Bandit as the HTTP server and implements the MCP JSON-RPC protocol directly to expose MIDI functionality as MCP tools.

**Target Users:** Musicians, sound designers, and developers who want to control MIDI hardware through natural language conversations with Claude AI.

## Development Commands

### Dependencies
```bash
mix deps.get              # Install dependencies
```

### Testing
```bash
mix test                  # Run all tests
mix test test/path_test.exs  # Run a specific test file
mix test test/path_test.exs:42  # Run test at specific line
```

### Code Formatting
```bash
mix format                # Format all Elixir files
mix format --check-formatted  # Check if files are formatted
```

### Building
```bash
mix compile               # Compile the project
```

### Running the MCP Server

**Development Mode:**
```bash
elixir run_mcp.exs        # Run the HTTP server (listens on port 3000)
PORT=8080 elixir run_mcp.exs  # Run on custom port
```

**Production Release:**
```bash
# Build a release for your current platform
MIX_ENV=prod mix release

# Run the release
_build/prod/rel/midi_ctrl/bin/midi_ctrl start

# Stop the release
_build/prod/rel/midi_ctrl/bin/midi_ctrl stop

# Run in daemon mode
_build/prod/rel/midi_ctrl/bin/midi_ctrl daemon
```

## Architecture

### Core Structure

The project implements an HTTP-based MCP server with three main modules:

- **`MIDICtrl.Router`** (lib/midi_ctrl/router.ex): The main HTTP router using Plug. Handles MCP JSON-RPC requests at `/mcp` endpoint and implements the MCP protocol methods (initialize, tools/list, tools/call, resources/list, resources/read). This is the entry point for the MCP server.

- **`MIDIOps`** (lib/midi_ops.ex): Contains MIDI operation implementations. Provides functions for:
  - `list_ports/0`: Query available MIDI ports
  - `send_cc_batch/4`: Send multiple CC changes with optional delays
  - `set_oscillator/3`: Switch MicroFreak oscillator types by name

- **`run_mcp.exs`**: Startup script that loads dependencies, starts the Bandit HTTP server on port 3000 (configurable via PORT env var), and keeps the server running.

### Key Dependencies

- **midiex** (~> 0.6.3): Elixir wrapper for MIDI functionality using Rust NIFs, providing access to MIDI ports and devices
- **bandit** (~> 1.8): Fast HTTP server built on Thousand Island
- **plug** (~> 1.18): Composable web middleware for HTTP request handling
- **jason** (~> 1.4): High-performance JSON encoding/decoding

### MCP Protocol Implementation

The server implements the MCP protocol via HTTP/JSON-RPC:

1. **HTTP Transport**: Uses Bandit HTTP server with Plug router
2. **JSON-RPC 2.0**: Handles MCP methods as JSON-RPC 2.0 requests
3. **MCP Methods Supported**:
   - `initialize`: Returns server capabilities and protocol version (2024-11-05)
   - `notifications/initialized`: Acknowledges initialization
   - `tools/list`: Returns available MIDI tools (list_ports, microfreak_cc, microfreak_set_oscillator)
   - `tools/call`: Executes tool requests with argument validation
   - `resources/list`: Lists available documentation resources
   - `resources/read`: Returns documentation content (MicroFreak MIDI reference)
   - `notifications/cancelled`: Handles cancellation notifications

### Request Flow

```
HTTP POST /mcp
  ↓
Plug.Parsers (extract JSON body)
  ↓
MIDICtrl.Router.call/2
  ↓
handle_mcp_method/3 (pattern match on method name)
  ↓
MIDIOps functions (MIDI operations)
  ↓
Midiex (Rust NIF → MIDI device)
  ↓
JSON-RPC 2.0 response
```

### Claude Desktop Integration

The server integrates with Claude Desktop via `mcp-remote`:

**Configuration file location:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

**Config:**
```json
{
  "mcpServers": {
    "midi_ctrl": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "http://localhost:3000/mcp"]
    }
  }
}
```

The `mcp-remote` npm package bridges stdio (Claude Desktop's native protocol) to HTTP (MIDICtrl server).

### Adding New MCP Tools

To add new MCP tools:

1. **Add tool definition** in `MIDICtrl.Router.handle_mcp_method("tools/list", ...)` (lib/midi_ctrl/router.ex:51-172)
   ```elixir
   %{
     name: "your_tool_name",
     description: "Clear description of what the tool does",
     inputSchema: %{
       type: "object",
       properties: %{
         your_param: %{type: "string", description: "Param description"}
       },
       required: ["your_param"]
     }
   }
   ```

2. **Implement tool execution** in `MIDICtrl.Router.handle_mcp_method("tools/call", ...)` (lib/midi_ctrl/router.ex:193+)
   ```elixir
   defp handle_mcp_method("tools/call", id, %{"name" => "your_tool_name", "arguments" => args}) do
     # Validate arguments
     # Call MIDIOps function
     # Return success/error response
   end
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanassound/midi_ctrl](https://github.com/nanassound/midi_ctrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
