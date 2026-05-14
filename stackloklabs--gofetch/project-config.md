---
trigger: always_on
description: This document provides essential context for Claude agents working with the GoFetch MCP (Model Context Protocol) server repository.
---

# Claude Agent Context - GoFetch MCP Server

This document provides essential context for Claude agents working with the GoFetch MCP (Model Context Protocol) server repository.

## Project Overview

**GoFetch** is a Go implementation of an MCP server that retrieves web content. It's designed as a more efficient alternative to the original Python MCP fetch server, offering:

- **Lower memory usage** and **faster startup/shutdown**
- **Single binary deployment** for enhanced security
- **Better concurrent request handling**
- **Container security** with non-root user and distroless images
- **Multiple transport protocols** (SSE and StreamableHTTP)

## Architecture & Key Components

### Core Packages

1. **`pkg/config/`** - Server configuration management
   - Handles command-line flags and environment variables
   - Supports transport types: `sse` and `streamable-http`
   - Configuration for port, user agent, robots.txt compliance, proxy settings

2. **`pkg/server/`** - MCP server implementation
   - Main server logic with transport-specific handlers
   - Tool registration and request handling
   - Endpoint management for SSE and StreamableHTTP

3. **`pkg/fetcher/`** - HTTP content retrieval
   - Handles HTTP requests with proper headers
   - Integrates with robots.txt checking
   - Content processing and error handling

4. **`pkg/processor/`** - Content processing
   - HTML to Markdown conversion using `html-to-markdown`
   - Content extraction using `go-readability`
   - Text formatting with pagination and truncation

5. **`pkg/robots/`** - Robots.txt compliance
   - Fetches and parses robots.txt files
   - Validates URL access permissions
   - Configurable to ignore robots.txt rules

### Entry Point
- **`cmd/server/main.go`** - Application entry point
  - Parses configuration
  - Creates and starts the fetch server
  - Handles graceful shutdown

## MCP Tool: `fetch`

The server provides a single MCP tool called `fetch` with these parameters:

```json
{
  "name": "fetch",
  "arguments": {
    "url": "https://example.com",           // Required: URL to fetch
    "max_length": 5000,                     // Optional: Max characters (default: 5000, max: 1000000)
    "start_index": 0,                       // Optional: Starting character index (default: 0)
    "raw": false                            // Optional: Return raw HTML vs markdown (default: false)
  }
}
```

## Transport Protocols

### 1. StreamableHTTP (Default - Recommended)
- **Endpoint**: `http://localhost:8080/mcp`
- **Session Management**: HTTP headers (`Mcp-Session-Id`)
- **Communication**: Single endpoint for both streaming and commands
- **Modern**: Preferred transport for new implementations

### 2. SSE (Legacy)
- **SSE Endpoint**: `http://localhost:8080/sse` (server-to-client)
- **Messages Endpoint**: `http://localhost:8080/messages` (client-to-server)
- **Session Management**: Query parameters (`?sessionid=...`)
- **Legacy**: Maintained for backward compatibility

## Configuration Options

### Command Line Flags
```bash
--transport string          # Transport type: sse or streamable-http (default: streamable-http)
--port int                  # Port number (default: 8080)
--user-agent string         # Custom User-Agent string
--ignore-robots-txt         # Ignore robots.txt rules
--proxy-url string          # Proxy URL for requests
```

### Environment Variables
```bash
TRANSPORT=sse               # Override transport type
MCP_PORT=8080              # Override port number
```

## Dependencies & Key Libraries

### Core Dependencies
- **`github.com/modelcontextprotocol/go-sdk`** v1.0.0 - MCP SDK for Go (stable release)
- **`github.com/JohannesKaufmann/html-to-markdown`** - HTML to Markdown conversion
- **`github.com/go-shiori/go-readability`** - Content extraction
- **`golang.org/x/net`** - HTTP client and HTML parsing

### Build & Development
- **Go 1.24+** required
- **Task** for build automation (see `Taskfile.yml`)
- **golangci-lint** for code quality
- **ko** for container image building

## Development Workflow

### Build Commands
```bash
task build          # Build the application
task run            # Run the application
task test           # Run tests
task test-integration # Run integration tests
task lint           # Run linting
task fmt            # Format code
task clean          # Clean build directory
```

### Testing
- Unit tests in each package (`*_test.go` files)
- Integration tests in `test/` directory
  - `test/integration-test.sh` - Tests SSE and StreamableHTTP transports with tool calls
  - `test/integration-endpoints.sh` - Tests endpoint accessibility and responses
- Requires `yardstick-client` for integration tests (installed via `go install`)

## Container Deployment

### Container Image
- **Registry**: `ghcr.io/stackloklabs/gofetch/server`
- **Security**: Non-root user, distroless image
- **Signing**: Container signing with build provenance

## API Usage Examples

### StreamableHTTP Example
```bash
# Initialize session
SESSION_ID=$(curl -s -D /dev/stderr -X POST "http://localhost:8080/mcp" \
  -H "Content-Type: application/json" \
  -H "Mcp-Protocol-Version: 2025-06-18" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StacklokLabs/gofetch](https://github.com/StacklokLabs/gofetch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
