---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building
```bash
# Build all packages
go build -v ./...

# Build the main binary
go build -o osmmcp ./cmd/osmmcp

# Build with version information
VERSION=$(git describe --tags --always)
COMMIT=$(git rev-parse --short HEAD)
BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
go build -ldflags="-X main.buildVersion=$VERSION -X main.buildCommit=$COMMIT -X main.buildDate=$BUILD_DATE" -o osmmcp ./cmd/osmmcp
```

### Running the Server

#### Stdio Transport (Default)
```bash
# Run with stdio transport for Claude Desktop integration
./osmmcp

# Run with debug logging
./osmmcp --debug
```

#### HTTP+SSE Transport (Anthropic API Integration)
```bash
# Enable HTTP+SSE dual transport for remote MCP clients
./osmmcp --enable-http --http-addr :7082

# With authentication
./osmmcp --enable-http --http-addr :7082 --http-auth-type bearer --http-auth-token your-secret

# With custom base URL for external access
./osmmcp --enable-http --http-addr :7082 --http-base-url https://your-domain.com
```

#### Monitoring and Health Checks
```bash
# Enable monitoring server (enabled by default)
./osmmcp --enable-monitoring --monitoring-addr :9090

# Disable monitoring
./osmmcp --enable-monitoring=false

# Run with both HTTP transport and monitoring
./osmmcp --enable-http --http-addr :7082 --enable-monitoring --monitoring-addr :9090
```

#### HTTP Transport Configuration
- `--enable-http`: Enable HTTP+SSE transport (disabled by default)
- `--http-addr`: HTTP server address (default: ":7082")
- `--http-base-url`: Base URL for service discovery (auto-detected if empty)
- `--http-auth-type`: Authentication type - "none", "bearer", or "basic" (default: "none")
- `--http-auth-token`: Authentication token/credentials

#### Monitoring Configuration
- `--enable-monitoring`: Enable Prometheus metrics and health endpoints (default: true)
- `--monitoring-addr`: Monitoring server address (default: ":9090")

#### Transport Endpoints
When HTTP transport is enabled, the following endpoints are available on port 7082:
- `GET /`: Service discovery with transport capabilities
- `GET /health`: Comprehensive health check with connection status
- `GET /ready`: Kubernetes-style readiness probe
- `GET /live`: Kubernetes-style liveness probe
- `GET /sse`: Server-Sent Events endpoint for MCP communication
- `POST /message`: JSON-RPC message endpoint (requires sessionId from SSE)
- `GET /sse/debug`: Debug information for SSE endpoint
- `GET /message/debug`: Debug information for message endpoint

#### Monitoring Endpoints
When monitoring is enabled, Prometheus metrics are available on port 9090:
- `GET /metrics`: Prometheus metrics endpoint (port 9090)

### Testing
```bash
# Run all tests
go test -v ./...

# Run tests for a specific package
go test -v ./pkg/cache/...

# Run a single test
go test -v -run TestFunctionName ./pkg/tools/...

# Test HTTP transport functionality
go test -v ./pkg/server/ -run TestHTTPTransport

# Run comprehensive dual transport integration test
./test_dual_transport.sh
```

### Code Quality
```bash
# Format code
go fmt ./...

# Check for issues
go vet ./...

# Run linting (if golangci-lint is installed)
golangci-lint run ./...
```

## Architecture Overview

This is a Go implementation of the Model Context Protocol (MCP) server providing OpenStreetMap capabilities to LLMs. The architecture follows a layered approach with clear separation of concerns:

### Core Architecture Layers

1. **MCP Server Layer** (`pkg/server/`) - Handles MCP protocol communication and request routing
2. **Tools Layer** (`pkg/tools/`) - Implements 25 OSM-specific tools using a registry pattern
3. **Core Utilities** (`pkg/core/`) - Shared functionality including HTTP retry logic, validation, error handling, and service clients
4. **OSM Integration** (`pkg/osm/`) - OpenStreetMap API client with rate limiting and caching
5. **Caching Layer** (`pkg/cache/`) - LRU caching for API responses and tile resources
6. **Monitoring Layer** (`pkg/monitoring/`) - Prometheus metrics, health checking, and observability

### Key Design Patterns

- **Registry Pattern**: All tools are registered centrally in `pkg/tools/registry.go` for maintainability
- **Composable Tools**: Tools are designed as primitives that can be combined for complex workflows
- **Fluent Builders**: Query builders (e.g., Overpass) use fluent interfaces for readability
- **Service Pattern**: External APIs (OSRM, Nominatim, Overpass) are wrapped in service objects with consistent interfaces

### Adding New Tools

1. Implement the tool handler function in `pkg/tools/`
2. Add the tool definition to the registry in `pkg/tools/registry.go`
3. Follow the existing patterns for parameter validation and error handling using `pkg/core/` utilities

### External Service Integration

The server integrates with several OpenStreetMap services, all with built-in rate limiting:
- **Nominatim**: Geocoding and reverse geocoding (default: 1 RPS)
- **Overpass API**: OSM data queries (default: 1 RPS)
- **OSRM**: Routing calculations (default: 1 RPS)
- **OSM Tiles**: Map image generation

### Error Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NERVsystems/osmmcp](https://github.com/NERVsystems/osmmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
