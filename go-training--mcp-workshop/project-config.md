---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the MCP Workshop repository for learning Model Context Protocol (MCP) development with Go. The project demonstrates building MCP servers and clients across 5 progressive modules, from basic implementations to advanced features like OAuth, observability, and proxy servers.

## Architecture

The codebase follows a modular structure with shared packages and separate example modules:

### Core Architecture

- **pkg/core/**: Context management and OAuth storage interfaces
  - `core.go`: Context helpers for request IDs (via `uuid`) and auth token extraction from HTTP headers or environment
  - `store.go`: Store interface for OAuth authorization codes and client management
- **pkg/operation/**: Tool registration system categorizing tools as read/write operations
  - `operation.go`: `RegisterCommonTool()` and `RegisterAuthTool()` functions
  - `echo/`, `caculator/`, `token/`: Individual tool implementations
- **pkg/logger/**: Structured logging with slog
- **pkg/store/**: Store implementations for OAuth data with factory pattern
  - `memory.go`: In-memory store implementation with thread-safe operations
  - `redis.go`: Redis-backed persistent store using rueidis client
  - `factory.go`: Factory pattern for creating store instances (memory or redis)
- **Module structure**: Each numbered directory (01-05) contains a complete working example

### Key Components

- **MCPServer wrapper**: Wraps `github.com/mark3labs/mcp-go/server.MCPServer` with Gin HTTP integration
  - `ServeHTTP()`: Returns `StreamableHTTPServer` with 30s heartbeat and context injection
  - `ServeStdio()`: Stdio transport with context injection via `server.ServeStdio()`
- **Tool registration**: Tools categorized as read/write operations via `operation.Tool` struct
  - `RegisterRead()` / `RegisterWrite()` methods append to internal slices
  - `Tools()` returns combined slice for batch registration with `s.AddTools()`
- **Transport support**: Both stdio and HTTP with unified auth via Go context
  - HTTP: Extracts `Authorization` header via `core.AuthFromRequest()`
  - Stdio: Extracts `API_KEY` env var via `core.AuthFromEnv()`
- **Context propagation**: `RequestIDKey` and `AuthKey` custom types for type-safe context values
  - `core.WithRequestID()`: Generates UUID request ID
  - `core.TokenFromContext()`: Retrieves auth token from context
  - `core.LoggerFromCtx()`: Returns logger with request_id field

## Development Commands

### Building All Binaries

The Makefile automates building all server binaries and running tests:

```bash
# Build all binaries to bin/ directory
make

# Clean all built binaries
make clean

# Run all tests
make test

# Run tests with verbose output
make test-verbose

# Run tests with coverage report
make test-cover

# Run store package tests only
make test-store

# Show all available targets
make help
```

### Running Individual Modules

Each module can be run independently:

```bash
# Basic MCP server (stdio mode)
go run 01-basic-mcp/server.go

# HTTP mode with custom address
go run 01-basic-mcp/server.go -transport http -addr :8080

# Token passthrough example
go run 02-basic-token-passthrough/server.go -transport http

# OAuth MCP server with memory store (default)
go run 03-oauth-mcp/dcr/oauth-server/server.go -client_id=<id> -client_secret=<secret> -addr :8095

# OAuth MCP server with Redis store
go run 03-oauth-mcp/dcr/oauth-server/server.go -client_id=<id> -client_secret=<secret> -addr :8095 -store redis -redis-addr localhost:6379

# OAuth client example
go run 03-oauth-mcp/dcr/oauth-client/client.go

# Observability server
go run 04-observability/server.go -transport http
```

### Common Server Flags

Most servers support these flags:

- `-transport` or `-t`: Transport type (`stdio` or `http`)
- `-addr`: Address to listen on (varies by module: `:8080` for basic, `:8095` for OAuth)
- `-log-level`: Log level (DEBUG, INFO, WARN, ERROR) - defaults to DEBUG in dev, INFO in production

OAuth server additional flags:

- `-client_id`: OAuth 2.0 Client ID (required)
- `-client_secret`: OAuth 2.0 Client Secret (required)
- `-provider`: OAuth provider (github, gitea, gitlab)
- `-gitea-host`: Gitea host URL (default: `https://gitea.com`)
- `-gitlab-host`: GitLab host URL (default: `https://gitlab.com`)
- `-store`: Store type (`memory` or `redis`) - defaults to `memory`
- `-redis-addr`: Redis address (default: `localhost:6379`) - only used when `-store=redis`
- `-redis-password`: Redis password - only used when `-store=redis`
- `-redis-db`: Redis database number (default: 0) - only used when `-store=redis`

### Standard Go Commands

```bash
# Install dependencies
go mod tidy

# Run tests
go test ./...

# Run tests with short flag (skips integration tests)
go test ./... -short

# Format code
go fmt ./...

# Vet code
go vet ./...

# Generate mocks (requires mockgen)
make mock
# or directly:
go generate ./pkg/mocks/...
```

## MCP Configuration

The repository includes `mcp.json` in the root for MCP client integration:

```json
{
  "mcpServers": {
    "default-stdio-server": {
      "type": "stdio",
      "command": "mcp-server",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-training/mcp-workshop](https://github.com/go-training/mcp-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
