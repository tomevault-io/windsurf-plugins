---
trigger: always_on
description: This file contains configuration and commands for Claude Code to assist with this project.
---

# Claude Code Configuration

This file contains configuration and commands for Claude Code to assist with this project.

## Project Overview

mcp-server-dump is a Go-based command-line tool for extracting documentation from MCP (Model Context Protocol) servers. It connects to MCP servers via multiple transports (STDIO/command, SSE, and streamable HTTP) and dumps their capabilities, tools, resources, and prompts to Markdown, JSON, HTML, or PDF format. The tool includes tool calling functionality to execute MCP tools and document their results, frontmatter support for static site generator integration, and rich structured context support via external YAML/JSON configuration files.

## Development Commands

### Build Commands
```bash
# Build from new entry point
go build -o mcp-server-dump ./cmd/mcp-server-dump

# Or use make-style command for convenience
go build -o mcp-server-dump ./cmd/mcp-server-dump
```

### Test Commands
```bash
go test ./...
```

### Lint Commands
```bash
go fmt ./...
go vet ./...
```

### Dependencies
```bash
go mod tidy
go mod download
```

### Linting
```bash
# Install golangci-lint v2.4.0 (required for Go 1.26 support)
go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.4.0

# Run linter (may need to use $GOPATH/bin/golangci-lint if not in PATH)
golangci-lint run
```

## Key Files

### Main Entry Point
- `cmd/mcp-server-dump/main.go` - Minimal main function that delegates to internal packages

### Internal Packages (Private API)
- `internal/app/` - Application logic and CLI configuration
  - `cli.go` - Command line interface definition
  - `runner.go` - Main application logic and MCP client implementation
  - `version.go` - Version handling with build info detection
  - `templates.go` - Embedded template filesystem
  - `templates/` - Go template files for markdown output formatting
- `internal/transport/` - MCP transport implementations
  - `factory.go` - Transport factory with configuration
  - `header.go` - HTTP header middleware
  - `content_fix.go` - Content-type normalization for streamable transport
- `internal/formatter/` - Output formatters
  - `markdown.go` - Markdown formatting with templates
  - `json.go` - JSON output formatting
  - `html.go` - HTML output (converts markdown via Goldmark)
  - `pdf.go` - PDF generation using go-pdf/fpdf
  - `frontmatter.go` - YAML/TOML/JSON frontmatter generation
  - `utils.go` - Common formatting utilities
- `internal/model/` - Data structures
  - `server.go` - ServerInfo, Tool, Resource, Prompt, Capabilities structs

### Configuration Files
- `go.mod` - Go module dependencies
- `.gitignore` - Git ignore patterns for Go projects  
- `.goreleaser.yaml` - GoReleaser configuration for multi-platform builds
- `README.md` - Project documentation

## Dependencies

### Production Dependencies
- `github.com/modelcontextprotocol/go-sdk` - Official MCP Go SDK for client/server communication
- `github.com/alecthomas/kong` - Command line argument parsing library
- `github.com/yuin/goldmark` - Markdown to HTML converter with GitHub Flavored Markdown support
- `codeberg.org/go-pdf/fpdf` - Pure Go PDF generation library with bookmark support
- `gopkg.in/yaml.v2` - YAML parsing and generation for frontmatter and context file support

### Development Tools
- Go 1.26.0+ - Required Go version
- Standard Go toolchain (go fmt, go vet, go test)

## Usage Examples

### Basic Usage
```bash
# Connect to filesystem server (command transport)
mcp-server-dump npx @modelcontextprotocol/server-filesystem /Users/username/Documents

# Connect to custom Node.js server
mcp-server-dump node server.js --port 3000

# Connect via SSE transport with headers
mcp-server-dump -t sse --endpoint "http://localhost:3001/sse" -H "Authorization:Bearer token"

# Connect via streamable transport
mcp-server-dump -t streamable --endpoint "http://localhost:3001/stream"

# Disable table of contents in markdown output
mcp-server-dump --no-toc node server.js

# Generate HTML output
mcp-server-dump -f html node server.js

# Generate PDF output (requires output file)
mcp-server-dump -f pdf -o server-docs.pdf node server.js

# Output to JSON file
mcp-server-dump -f json -o output.json python mcp_server.py
```

### Tool Calling Usage
```bash
# Call a specific tool by name
mcp-server-dump --call-tool="get_weather" node server.js

# Call a tool with arguments
mcp-server-dump --call-tool="get_weather" --tool-args='{"location":"London"}' node server.js

# Call multiple specific tools
mcp-server-dump --call-tool="search" --call-tool="analyze" node server.js

# Call all available tools (for testing/documentation)
mcp-server-dump --call-all-tools node server.js

# Call all tools with specific arguments
mcp-server-dump --call-all-tools --tool-args='{"test":true}' node server.js
```

### Context Enhancement Usage
```bash
# Basic context usage with single file
mcp-server-dump --context-file context.yaml npx @modelcontextprotocol/server-filesystem /docs

# Multiple context files (merged in order)
mcp-server-dump --context-file base.yaml --context-file overrides.json node server.js

# Context with different output formats
mcp-server-dump --context-file context.yaml -f html -o docs.html node server.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SPANDigital/mcp-server-dump](https://github.com/SPANDigital/mcp-server-dump) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
