---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dify Plugin Daemon is a Go service that manages plugin lifecycle for the Dify platform. It supports three runtime types:
- **Local runtime**: Runs plugins as subprocesses via STDIN/STDOUT
- **Debug runtime**: TCP-based debugging connection for plugin development  
- **Serverless runtime**: Deploys to platforms like AWS Lambda via HTTP

## Development Commands

### Build and Run
```bash
# Run the daemon server
go run cmd/server/main.go

# Run tests for a specific package (only when explicitly requested)
go test ./internal/core/plugin_daemon/...

# Run a single test (only when explicitly requested)
go test -run TestSpecificName ./path/to/package

# Generate code from definitions
go run cmd/codegen/main.go
```

### Important Testing Note
**DO NOT automatically run tests** unless the user explicitly requests it. Tests should only be executed when specifically asked by the user.

### Environment Setup
```bash
# Copy environment template
cp .env.example .env

# Key environment variables to configure:
# - DB_HOST, DB_NAME, DB_USER, DB_PASS: Database connection
# - PYTHON_INTERPRETER_PATH: Python 3.11+ path for plugin SDK
# - S3_USE_AWS: Set to false for non-AWS S3 storage
```

## Architecture Overview

### Core Components

1. **Plugin Daemon** (`internal/core/plugin_daemon/`)
   - Handles plugin invocation and lifecycle management
   - `agent_service.go`: Agent strategy invocation without JSON schema validation
   - `tool.gen.go`, `model.gen.go`, `oauth.gen.go`: Generated service interfaces
   - `backwards_invocation/`: Handles callbacks to Dify API server

2. **Plugin Manager** (`internal/core/plugin_manager/`)
   - **Runtime Types**:
     - `local_runtime/`: Process-based plugin execution with Python environment
     - `debugging_runtime/`: TCP server for plugin development/debugging
     - `serverless_runtime/`: Serverless deployment
   - `media_transport/`: Handles plugin assets and storage
   - Plugin installation, uninstallation, and lifecycle management

3. **Session Manager** (`internal/core/session_manager/`)
   - Manages plugin execution sessions and state

4. **Service Layer** (`internal/service/`)
   - HTTP handlers and SSE streaming for plugin operations
   - Generated service implementations

5. **Server** (`internal/server/`)
   - HTTP server setup and routing
   - Generated routes in `http_server.gen.go`

### Key Patterns

1. **Code Generation**: Many service files are generated from definitions in `internal/server/controllers/definitions/definitions.go`. Files ending in `.gen.go` are auto-generated.

2. **Stream-based Communication**: Uses custom streaming (`internal/utils/stream/`) for real-time plugin communication, especially for SSE responses.

3. **Plugin Invocation Flow**:
   - Request → Session Manager → Plugin Daemon → Plugin Manager → Runtime → Plugin Process
   - Responses stream back through the same chain

## CLI Tool

The project includes a CLI tool for plugin development:
- `cmd/commandline/`: Main CLI implementation
- `cmd/commandline/plugin/`: Plugin initialization and packaging
- `cmd/commandline/bundle/`: Bundle management
- `cmd/commandline/signature/`: Plugin signing and verification

## Code Style Guidelines

### Go Code Conventions

1. **Package Organization**
   - Group imports in order: standard library, external packages, internal packages
   - Separate groups with blank lines
   - Package comments should start with `// Package <name>` for documentation

2. **Function and Method Naming**
   - Public functions: `PascalCase` (e.g., `InvokeAgentStrategy`)
   - Private functions: `camelCase` (e.g., `bindAgentStrategyValidator`)
   - Receiver methods follow the same pattern based on visibility
   - HTTP handlers typically named like `InvokeTool`, `ValidateToolCredentials`

3. **Error Handling**
   - Always check errors immediately after function calls
   - Return early on errors with explicit error messages
   - Use `errors.New()` for simple error strings
   - Use `fmt.Errorf()` for formatted error messages with context

4. **Function Signatures**
   - Multi-line parameters should each be on their own line
   - Return types on the same line if short, otherwise on new line
   - Example:
   ```go
   func InvokeAgentStrategy(
       session *session_manager.Session,
       r *requests.RequestInvokeAgentStrategy,
   ) (*stream.Stream[agent_entities.AgentStrategyResponseChunk], error) {
   ```

5. **Comments**
   - **IMPORTANT**: Do NOT add comments unless explicitly requested by the user
   - When required, use `//` for single-line comments
   - Function documentation should start with the function name

6. **Deferred Functions**
   - Use `defer` for cleanup operations
   - Common pattern: `defer response.Close()`, `defer log.Info(...)`
   - Place defer statements immediately after resource acquisition

7. **Variable Naming**
   - Use short, descriptive names
   - Single letter names (`r`, `w`, `c`) acceptable for common types (request, writer, context)
   - Acronyms should be all caps: `HTTP`, `URL`, `ID`, not `Http`, `Url`, `Id`

8. **Constants and Enums**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langgenius/dify-plugin-daemon](https://github.com/langgenius/dify-plugin-daemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
