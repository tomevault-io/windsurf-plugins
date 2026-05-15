---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a production-ready TypeScript template for building Model Context Protocol (MCP) servers and clients following the **MCP 2025-03-26 specification**. It provides a comprehensive foundation with built-in utilities, authentication, error handling, and service integrations.

**Key Stats:**

- 57 TypeScript source files
- 32 directories in src/
- Supports both stdio and HTTP (SSE) transports
- Built-in authentication (JWT/OAuth)
- Comprehensive error handling and logging
- Ready-to-use DuckDB and OpenRouter integrations

## Development Commands

### Build & Run

- `npm run build` - Build TypeScript and make executable
- `npm run rebuild` - Clean build (removes node_modules, logs, dist)
- `npm start` - Run the built server (stdio transport)
- `npm run start:stdio` - Run server with stdio transport and debug logging
- `npm run start:http` - Run server with HTTP transport and debug logging

### Development Tools

- `npm run format` - Format code with Prettier
- `npm run docs:generate` - Generate TypeDoc documentation
- `npm run tree` - Generate project structure tree
- `npm run inspector` - Launch MCP inspector for debugging
- `npm run depcheck` - Check for unused dependencies
- `npm run fetch-spec` - Fetch OpenAPI specification
- `npm run db:duckdb-example` - Run DuckDB example with debug logging

### Testing & Quality

**Important**: No specific test framework is configured. Before adding tests:

1. Check README or search codebase for testing approach
2. Ask user for preferred test framework if not found
3. Always run linting after code changes

## Project Architecture

### Core Components

**Entry Point (`src/index.ts`)**:

- Main application entry with graceful shutdown handling
- Initializes logger and configuration
- Starts appropriate transport (stdio/HTTP)
- Handles process signals and unhandled errors

**Configuration (`src/config/index.ts`)**:

- Environment variable validation with Zod schemas
- Project root detection and directory creation
- Comprehensive config object with defaults
- Supports development and production environments

**MCP Server (`src/mcp-server/`)**:

- `server.ts` - Core server initialization and transport selection
- `createMcpServerInstance()` - Factory for McpServer instances
- Per-session instances for HTTP, singleton for stdio
- Automatic tool and resource registration

**MCP Client (`src/mcp-client/`)**:

- Modular client implementation for connecting to external MCP servers
- `core/clientManager.ts` - Connection lifecycle management
- `client-config/configLoader.ts` - Configuration validation
- Support for both stdio and HTTP transports
- Connection caching and automatic reconnection

### Transport Layer

**Stdio Transport (`src/mcp-server/transports/stdioTransport.ts`)**:

- Single server instance communicating via stdin/stdout
- Simple process-based communication
- No authentication required

**HTTP Transport (`src/mcp-server/transports/httpTransport.ts`)**:

- Hono-based web server with Server-Sent Events
- Per-session MCP server instances
- CORS support and port retry logic
- Session management with in-memory store
- Rate limiting and security headers

**Authentication (`src/mcp-server/transports/auth/`)**:

- JWT mode: Self-contained tokens for development
- OAuth mode: External authorization server integration
- Middleware-based authentication
- Configurable auth strategies

### Utilities Framework (`src/utils/`)

**Error Handling (`src/utils/internal/errorHandler.ts`)**:

- Centralized `ErrorHandler` class with pattern-based classification
- Automatic error code determination
- Consistent logging and context tracking
- `tryCatch` helper for robust error handling
- Support for custom error mappings

**Logging (`src/utils/internal/logger.ts`)**:

- Winston-based structured logging
- File rotation and MCP notifications
- Context-aware logging with request correlation
- Multiple log levels and output formats

**Request Context (`src/utils/internal/requestContext.ts`)**:

- Request/operation tracking across the application
- Correlation ID generation and management
- Context inheritance and merging

**Security (`src/utils/security/`)**:

- Input sanitization and validation
- Rate limiting with configurable limits
- ID generation (UUIDs and prefixed IDs)
- Log redaction for sensitive data

**Parsing (`src/utils/parsing/`)**:

- Natural language date parsing with chrono-node
- Partial JSON parsing with error handling
- Think block extraction from LLM responses

**Metrics (`src/utils/metrics/`)**:

- Token counting with tiktoken
- Performance and usage tracking utilities

**Network (`src/utils/network/`)**:

- HTTP requests with timeout support
- Fetch utilities with error handling

### Services Layer (`src/services/`)

**DuckDB Service (`src/services/duck-db/`)**:

- Complete DuckDB integration with connection management
- Query execution, streaming, and transaction support
- Extension loading and prepared statements
- Type-safe query interfaces

**OpenRouter Provider (`src/services/llm-providers/openRouterProvider.ts`)**:

- LLM integration via OpenRouter API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joho/pocketsmith-mcp](https://github.com/joho/pocketsmith-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
