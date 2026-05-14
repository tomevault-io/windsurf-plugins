---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Niffler is an AI-powered terminal assistant written in Nim. It provides a conversational interface to interact with AI models while supporting tool calling for file operations, command execution, and web fetching.

## Build and Development Commands

### Building
```bash
# Development build
nim c src/niffler.nim

# Optimized release build  
nimble build
```

### Testing
```bash
# Run all tests
nimble test

# Run individual test suites
nim c -r test_tool_calling.nim
nim c -r test_api_integration.nim
nim c -r test_tool_execution.nim
```

### Dependencies
```bash
# Install dependencies, they are all listed in niffler.nimble
nimble install sunny
```

## Architecture

### Core Components

**Thread-based Architecture**: The application uses a multi-threaded design with dedicated workers:
- **API Worker** (`src/api/api.nim`): Handles LLM communication and tool calling orchestration
- **Tool Worker** (`src/tools/worker.nim`): Executes individual tools with validation
- **Main Thread**: Manages UI and coordinates between workers via channels

**Channel Communication** (`src/core/channels.nim`): Thread-safe message passing between workers using Nim's channels for:
- API requests/responses
- Tool execution requests/results  
- UI updates and shutdown signals

**Tool System** (`src/tools/`):
- **Schema-based validation** (`schemas.nim`): JSON Schema validation for tool parameters
- **Registry pattern** (`registry.nim`): Central tool registration and lookup
- **Six core tools**: bash, read, list, edit, create, fetch
- **Security features**: Path sanitization, timeout enforcement, confirmation requirements

**Message Flow**:
1. User input → Main thread
2. Main thread → API worker (via channels)
3. API worker → LLM API → Tool calls detected
4. API worker → Tool worker (tool execution)
5. Tool worker → File system/commands → Results back to API worker
6. API worker → LLM API (continue conversation with tool results)
7. Final response → Main thread → User display

### Key Files

- `src/niffler.nim`: CLI entry point with docopt command dispatch
- `src/core/app.nim`: Application lifecycle and coordination
- `src/api/api.nim`: LLM API integration with tool calling support
- `src/api/http_client.nim`: OpenAI-compatible HTTP client
- `src/tools/worker.nim`: Tool execution engine
- `src/types/messages.nim`: Message type definitions for LLM and tool communication
- `src/ui/cli.nim`: Interactive terminal interface

### Configuration

Configuration system (`src/core/config.nim`):
- Model definitions with nicknames, base URLs, and API keys
- Environment variable support for API keys
- Config file location: `~/.niffler/config.yaml`

### Database Access

Niffler uses TiDB (MySQL-compatible) for persistent storage of conversations, messages, and token usage data.

**Database Configuration**: See `~/.niffler/config.yaml` for database connection settings (host, port, database name, etc.)

**Direct Access**:
```bash
# Connect to database for queries (using MySQL client)
mysql -h 127.0.0.1 -P 4000 -u root niffler

# View database schema
mysql -h 127.0.0.1 -P 4000 -u root niffler -e "SHOW TABLES"
mysql -h 127.0.0.1 -P 4000 -u root niffler -e "DESCRIBE conversation"

# Example queries
mysql -h 127.0.0.1 -P 4000 -u root niffler -e "SELECT * FROM conversation ORDER BY created_at DESC LIMIT 5"
mysql -h 127.0.0.1 -P 4000 -u root niffler -e "SELECT created_at, model, input_tokens, output_tokens, total_cost FROM model_token_usage ORDER BY created_at DESC LIMIT 10"
```

**Key Tables**:
- `conversation`: Conversation metadata and settings
- `conversation_message`: Individual messages in conversations
- `model_token_usage`: Token usage and cost tracking per API call
- `conversation_thinking_token`: Reasoning/thinking token storage

**Debugging Tips**:
- Use `SHOW TABLES` to list all tables in TiDB
- Token costs are tracked per API call with ISO timestamp format (`2025-01-15T14:30:45`)
- Session costs use `WHERE created_at >= ?` with current app start time for filtering

## Tool Calling Implementation

The tool calling system follows OpenAI's function calling specification:
- Tool schemas are JSON Schema definitions
- Tool calls are validated before execution
- Tools requiring confirmation: bash, edit, create (dangerous operations)
- Tools skipping confirmation: read, list, fetch (safe operations)
- Multi-turn conversations supported with tool result integration

## Dependencies and Libraries

- **docopt**: Command line argument parsing
- **sunny**: JSON handling

## Development Notes

- All compilation requires `--threads:on -d:ssl` flags (set in config.nims)
- Thread safety is critical - use channels for inter-thread communication
- Tool implementations must validate arguments against their schemas
- Security is paramount - all file paths are sanitized and validated
- The codebase follows Nim naming conventions and coding style

## Nim Coding Guidelines

### Code Style and Conventions
- Use camelCase, not snake_case (avoid `_` in naming)
- Do not shadow the local `result` variable (Nim built-in)
- Doc comments: `##` below proc signature

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gokr/niffler](https://github.com/gokr/niffler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
