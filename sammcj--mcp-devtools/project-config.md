---
trigger: always_on
description: **MCP DevTools** is a single, high-performance MCP (Model Context Protocol) server written in Go that replaces many Node.js and Python-based MCP servers with one efficient binary. It provides access to essential developer tools through a unified, modular interface that can be easily extended with new tools.
---

# GitHub Copilot Instructions for MCP DevTools

## Project Overview

**MCP DevTools** is a single, high-performance MCP (Model Context Protocol) server written in Go that replaces many Node.js and Python-based MCP servers with one efficient binary. It provides access to essential developer tools through a unified, modular interface that can be easily extended with new tools.

**Key Features:**
- Single binary solution replacing multiple resource-heavy servers
- 16+ essential developer tools in one package
- Built in Go for speed, efficiency, and minimal memory footprint
- Fast startup and response times
- Modular tool registry architecture allowing easy addition of new tools
- Supports multiple transports: stdio (default) and streamable HTTP

## Development Setup

### Building the Project

```bash
# Build the server
make build

# The binary will be created at: bin/mcp-devtools
```

### Running the Server

```bash
# Run with stdio transport (default)
make run

# Run with HTTP transport
make run-http
```

### Testing

```bash
# Run all tests (includes external API integration tests, ~10s)
make test

# Run fast tests (skips external dependencies, ~7s)
make test-fast

# Run tests with detailed timing
make test-verbose
```

### Linting and Code Quality

```bash
# Format code
make fmt

# Run linters and modernisation checks
make lint
# This runs: gofmt, golangci-lint, and gopls modernize
```

### Dependencies

```bash
# Install Go dependencies
make deps

# Install all dependencies (Go + Python for document processing)
make install-all
```

## Project Structure

```
mcp-devtools/
├── internal/
│   ├── tools/           # All tool implementations
│   ├── registry/        # Tool registration system
│   ├── security/        # Security framework for file/network operations
│   ├── handlers/        # MCP protocol handlers
│   ├── config/          # Configuration management
│   ├── oauth/           # OAuth functionality
│   ├── cache/           # Caching utilities
│   ├── utils/           # Utility functions
│   └── imports/         # Import management
├── tests/
│   ├── tools/           # Unit tests for tools (REQUIRED for all tools)
│   ├── benchmarks/      # Performance and token cost tests
│   ├── oauth/           # OAuth tests
│   └── unit/            # Unit tests for internal packages
├── docs/
│   └── tools/           # Tool documentation (REQUIRED when adding tools)
├── main.go              # Entry point - import new tools here
├── Makefile             # Build, test, and development commands
└── mcp.json             # MCP server configuration
```

## Contribution Guidelines

### Before Committing

1. **Format your code:** `make fmt`
2. **Run linters:** `make lint` (must pass without errors)
3. **Run tests:** `make test-fast` (must pass all tests)
4. **Build successfully:** `make build` (must compile without errors)

### Code Standards

- Follow Go best practices and idiomatic patterns
- Use British English spelling throughout code and documentation
- No marketing terms like "comprehensive" or "production-grade"
- Focus on clear, concise, actionable technical guidance
- Keep responses token-efficient (avoid returning unnecessary data)

### Adding New Tools

1. Create package under `internal/tools/[category]/[toolname]/`
2. Implement `tools.Tool` interface with `Definition()` and `Execute()` methods
3. Register tool in `init()` function using `registry.Register(&YourTool{})`
4. Import the package in `internal/imports/tools.go` (NOT in `main.go`)
5. Add unit tests in `tests/tools/` directory
6. Add documentation in `docs/tools/` with clear, concise information
7. Update `docs/tools/overview.md`
8. Integrate with security framework if accessing files/URLs
9. Verify token cost with `ENABLE_ADDITIONAL_TOOLS=your_tool_name make benchmark-tokens`

**Important**: Do NOT add tool imports directly to `main.go`. Use the imports registry system in `internal/imports/tools.go` instead.

## Architecture & Structure

This is a modular MCP (Model Context Protocol) server written in Go with a tool registry architecture. Each tool implements the `tools.Tool` interface and registers itself through `internal/registry/`. The main server supports multiple transports (stdio, streamable HTTP).

## ⚠️ CRITICAL: stdio Mode Logging Violations

**MOST IMPORTANT CHECK IN EVERY REVIEW:**

When the server runs in stdio mode (default transport), ANY output to stdout/stderr will break the MCP protocol and cause catastrophic failures. This is the #1 bug to prevent.

### What to Check in EVERY Pull Request:
1. **No direct stdout/stderr writes:**
   - ❌ NEVER: `fmt.Println()`, `fmt.Printf()`, `log.Println()`, `fmt.Fprintf(os.Stdout, ...)`
   - ❌ NEVER: `os.Stdout.Write()`, `os.Stderr.Write()`, `print()`, `println()`
   - ✅ ALWAYS: Use `logger.Info()`, `logger.Debug()`, `logger.Error()`, etc.

2. **No external commands that write to stdout/stderr in stdio mode:**
   - Check all `exec.Command()` calls
   - Ensure stdout/stderr are captured or redirected when server is in stdio mode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sammcj/mcp-devtools](https://github.com/sammcj/mcp-devtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
