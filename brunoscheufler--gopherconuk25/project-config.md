---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GopherCon UK 2025 presentation project focused on "Building a framework for reliable data migrations in Go". The codebase demonstrates _Notely_, an example application used to learn about different migration strategies in Go, including:

- **Migration from legacy data stores**: Moving existing notes from legacy systems to new, more efficient databases without downtime
- **Horizontal sharding**: Implementing sharding strategies to distribute notes across multiple databases for scalability
- **Zero-downtime deployments**: Using data proxies and rolling releases to maintain service availability during migrations

The application uses SQLite for data persistence and includes a comprehensive telemetry system for monitoring migration progress and system performance.

## Architecture

### Core Components

- **Account System**: User account management with UUID-based identification
- **Note System**: Content management system where users can create, read, update, and delete notes with timestamps
- **Data Proxy**: Decoupled service mediating data store access with JSON RPC interface for zero-downtime migrations
- **Storage Layer**: SQLite-based persistence using the `modernc.org/sqlite` driver (pure Go implementation)
- **REST API**: HTTP server with full CRUD operations for accounts and notes
- **CLI Interface**: Terminal User Interface (TUI) using `github.com/rivo/tview` for interactive management and deployment control
- **Load Generator**: Simulator for generating realistic user activity and testing migration scenarios
- **Telemetry System**: Centralized logging and statistics collection with live monitoring of proxy access and shard metrics
- **Deployment Controller**: Rolling release management for data proxy instances with automated health checks

### Key Patterns

- **Modular Architecture**: Clean separation of concerns with dedicated packages (`store/`, `cli/`, `telemetry/`, `proxy/`, `util/`)
- **Interface-Driven Design**: Both `AccountStore` and `NoteStore` are defined as interfaces, enabling proxy patterns and alternative implementations
- **Context-Aware Operations**: All store operations accept `context.Context` for cancellation and timeout handling
- **UUID Identifiers**: Uses `github.com/google/uuid` for all entity identification
- **Data Directory Pattern**: Creates a `.data/` directory in the working directory for SQLite database files
- **Graceful Shutdown**: HTTP server supports graceful shutdown with signal handling
- **Multi-Mode Operation**: Can run as HTTP-only server, CLI interface, or data proxy mode
- **Health Check Validation**: CLI mode validates server health before launching interface
- **Port Availability Check**: Verifies port is free before initialization to fail fast
- **Process Management**: Child process spawning for data proxies with log capture and health monitoring
- **Rolling Deployments**: Deployment controller manages multiple proxy instances for zero-downtime updates
- **Retry Logic**: Configurable retry mechanisms with error matching for resilient operations

## Development Commands

### Building and Running
```bash
go run .                    # Run HTTP server only
go run . -cli               # Run HTTP server + CLI interface
go run . -cli -theme=light  # Run with light theme
go run . --port=3000        # Run on custom port
go run . -cli --port=3000   # Run CLI mode on custom port

# Data proxy mode
go run . --proxy --proxy-port=9001  # Run as data proxy on port 9001

# Load generator mode
go run . --gen              # Enable load generator with defaults
go run . --gen --concurrency=10 --notes-per-account=5 --rpm=120  # Custom load parameters
go run . -cli --gen         # CLI mode with load generator

# Combined modes
go run . -cli --gen --theme=light --port=3000  # CLI + load generator + custom theme/port

go build -o app .          # Build binary
```

### Testing
```bash
go test ./...              # Run all tests
go test -v ./...           # Run tests with verbose output
```

### Dependencies
```bash
go mod tidy               # Clean up dependencies
go mod download           # Download dependencies
```

## Code Style Guidelines

### Early Returns

**Always prefer early returns over if/else chains** to improve code readability and reduce nesting:

**✅ Good - Early Return:**
```go
func validateUser(user User) error {
    if user.ID == "" {
        return errors.New("user ID is required")
    }
    if user.Name == "" {
        return errors.New("user name is required")
    }
    return nil
}

func handleRequest(w http.ResponseWriter, r *http.Request) {
    if err := validateInput(r); err != nil {
        writeError(w, http.StatusBadRequest, err.Error())
        return
    }
    if !isAuthorized(r) {
        writeError(w, http.StatusUnauthorized, "Unauthorized")
        return
    }
    // Happy path logic here
    processRequest(w, r)
}
```

**❌ Avoid - if/else chains:**
```go
func validateUser(user User) error {
    if user.ID == "" {
        return errors.New("user ID is required")
    } else {
        if user.Name == "" {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrunoScheufler/GopherConUK25](https://github.com/BrunoScheufler/GopherConUK25) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
