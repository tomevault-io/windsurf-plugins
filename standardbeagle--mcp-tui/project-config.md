---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Recent Updates (2025-01-12) - v0.2.0 Release

This project has implemented revolutionary UI improvements making MCP-TUI significantly more user-friendly and practical.

### Major Features Added
- **Tabbed Navigation System**: Visual tabs with arrow key navigation between Saved/Discovery/Manual modes
- **File Discovery Engine**: Automatically finds Claude Desktop, VS Code MCP, and MCP-TUI configuration files
- **Combined Command Input**: Default single-line input for commands like "brum --mcp" (press 'C' to toggle)
- **Enhanced Connection Management**: Visual connection cards with server enumeration and descriptions
- **Smart Input Priority**: Form fields take precedence over UI navigation keys

### Key Technical Improvements
- **MCP Validation**: Only shows JSON files with actual MCP server configurations
- **Multi-Format Support**: Compatible with Claude Desktop, VS Code MCP, and native formats
- **Enhanced Navigation**: Fixed focus issues and improved keyboard navigation throughout
- **Server Enumeration**: Display individual server names and descriptions from discovered files

### Key Files Modified
- `internal/tui/models/connections.go` - Enhanced with file discovery and server enumeration
- `internal/tui/screens/connection.go` - Revolutionary tabbed interface and navigation
- `internal/tui/app/manager.go` - Auto-connect functionality 
- `internal/tui/screens/main.go` - Navigation focus fixes
- `examples/` - Comprehensive configuration examples

## Project Overview

MCP-TUI is a Go-based test client for Model Context Protocol (MCP) servers that provides both an interactive Terminal User Interface (TUI) mode and a scriptable Command Line Interface (CLI) mode. It supports multiple transport types (stdio, SSE, HTTP) and allows users to browse and interact with MCP servers, execute tools, resources, and prompts.

## Development Commands

### Build
```bash
# Build the binary
go build -o mcp-tui .

# Build and install to ~/.local/bin
./build.sh
```

### Run Tests
```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run specific test
go test -run TestName
```

### Lint
```bash
# Install golangci-lint if not present
go install github.com/golangci-lint/golangci-lint/cmd/golangci-lint@latest

# Run linter
golangci-lint run

# Format code
go fmt ./...
```

## Architecture

### Core Components

1. **Transport Layer** - Handles different connection types:
   - `client.go` - Creates MCP clients for stdio, SSE, and HTTP transports
   - Platform-specific process management in `process_unix.go` and `process_windows.go`

2. **UI Layer** - Terminal interface implementation:
   - `tui.go` - Main TUI implementation using bubbletea framework
   - Handles connection management, tool execution, and result display
   - Supports scrollable output and progress tracking

3. **CLI Layer** - Command-line interface:
   - `main.go` - Entry point with cobra command definitions
   - `cmd_*.go` files implement subcommands:
     - `cmd_tool.go` - Tool listing, description, and execution
     - `cmd_resource.go` - Resource listing and reading
     - `cmd_prompt.go` - Prompt listing and retrieval

### Key Design Patterns

- **Platform Abstraction**: Build tags separate Unix and Windows implementations for process and signal handling
- **Command Pattern**: CLI commands follow cobra's command pattern with consistent connection handling
- **State Management**: TUI uses bubbletea's Elm-style architecture for state updates
- **Type Conversion**: Automatic conversion of CLI string inputs to JSON schema types in tool execution

### Dependencies

The project uses:
- `github.com/modelcontextprotocol/go-sdk` for official MCP protocol implementation (v0.2.0)
- `github.com/charmbracelet/bubbletea` for terminal UI
- `github.com/spf13/cobra` for CLI framework
- `github.com/atotto/clipboard` for clipboard support

## MCP Transport Implementation Knowledge

### Current Transport Support Status

**✅ STDIO Transport**: Fully working and recommended
- Uses `officialMCP.NewCommandTransport(cmd)` 
- Includes command validation security (`configPkg.ValidateCommand`)
- Handles process management correctly across platforms

**✅ HTTP Transport**: Working for request/response patterns
- Uses `officialMCP.NewStreamableClientTransport` 
- Good for API-style interactions
- Synchronous request/response model

**✅ SSE Transport**: Working with critical context fix
- Uses `officialMCP.NewSSEClientTransport`
- **CRITICAL**: Requires `context.Background()` for connection, not CLI timeout context
- Implements hanging GET + POST pattern per MCP spec:
  1. GET /sse → establish SSE stream and get sessionId
  2. POST to session endpoint → send MCP requests (get 202 Accepted)
  3. Responses arrive via SSE stream
- **Custom HTTP client needed**: No timeout for SSE streams

### MCP Protocol Understanding

**HTTP Transport Specification (2025-06-18)**:
- POST JSON-RPC messages → Server returns 202 Accepted
- Server can respond with:
  - `Content-Type: application/json` (direct response)
  - `Content-Type: text/event-stream` (SSE stream response)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [standardbeagle/mcp-tui](https://github.com/standardbeagle/mcp-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
