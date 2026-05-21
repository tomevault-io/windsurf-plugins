---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mule is an AI workflow platform that enables users to create, configure, and execute complex AI-powered workflows. It combines AI agents powered by **pi RPC**, custom skills, and WebAssembly modules to create flexible automation pipelines, exposed through an OpenAI-compatible API.

The project has been migrated from Google ADK to pi RPC for agent execution, providing a more flexible and extensible architecture with skills support.

## Development Commands

### Building and Running
```bash
# Build the application
make build
# or manually: cd cmd/api && CGO_ENABLED=1 GOOS=linux go build -o bin/mule

# Run the server (requires PostgreSQL)
./cmd/api/bin/mule -db "postgres://mule:mule@localhost:5432/mulev2?sslmode=disable"

# Run with hot reload during development
make air

# Run with Docker
docker-compose up -d

# Run staging environment
docker-compose -f docker-compose.staging.yml up -d
```

### Testing and Code Quality
```bash
# Run all tests
make test

# Run linting
make lint

# Format code
make fmt

# Run a single test
go test -v ./internal/engine -run TestEngineExecuteWorkflow

# Run tests for specific package
go test -v ./internal/agent/pirc/...
```

### Development Setup
```bash
# Set up test data (providers, agents, skills, WASM modules)
./setup-dev.sh

# Test workflow execution
./test_workflow.sh
```

## Architecture

### Core Primitives

The system is built around six core primitives stored in PostgreSQL:

1. **Providers** - AI provider configurations (OpenAI-compatible APIs)
   - Table: `providers`
   - Configuration: API base URL, encrypted API key
   - Supports: Anthropic, OpenAI, Google providers
   - Enables dynamic model discovery

2. **Skills** - Pi agent skills that can be assigned to agents
   - Table: `skills`
   - Stores skill name, description, path (directory), and enabled status
   - Skills are bound to agents via `agent_skills` junction table
   - Pi skills provide extensibility (file operations, grep, find, bash, read, write, edit, etc.)

3. **Agents** - AI agents powered by pi RPC runtime
   - Table: `agents`
   - References: provider_id, model_id, system_prompt, pi_config (JSONB)
   - Uses pi RPC for execution with configurable skills, thinking level, tools, and extensions

4. **WASM Modules** - WebAssembly modules for imperative code
   - Table: `wasm_modules`
   - Executed using wazero library in sandboxed environment
   - Binary module_data stored in database

5. **Workflows** - Ordered sequences of workflow steps
   - Table: `workflows`
   - Can be synchronous or asynchronous (`is_async` flag)
   - Can be invoked via `/v1/chat/completions` with model prefix `workflow/` or `async/workflow/`

6. **Workflow Steps** - Individual execution steps
   - Table: `workflow_steps`
   - Types: "AGENT" (invokes agent via pi RPC) or "WASM" (executes WASM)
   - Ordered by `step_order` within a workflow

### Execution Flow

1. **Configuration Phase**: Primitives configured via UI/API → Stored in PostgreSQL via Primitive Manager
2. **Execution Phase**: User calls `/v1/chat/completions` → Request queued as Job → Worker executes Workflow Steps → Each step invokes Agent (pi RPC) or WASM (wazero) → Results streamed via WebSocket

### Key Components

- **cmd/api/**: Main application entry point, HTTP handlers, API server
  - `server.go`: HTTP server setup and routing
  - `handlers.go`: OpenAI-compatible API endpoints (`/v1`) and management APIs
  - `memory_handlers.go`: Memory and semantic search endpoints
  - `wasm_handlers.go`: WASM module management endpoints
  - `integration_test.go`, `comprehensive_test.go`, `skills_test.go`: API tests

- **internal/**: Core application logic
  - `agent/`: Agent runtime with pi RPC integration
  - `agent/pirc/`: **pi RPC bridge package** - handles subprocess management and event streaming
    - `pibridge.go`: PI subprocess management, RPC command execution, event parsing
    - `event_mapper.go`: Converts pi events to Mule WebSocket format
    - `websocket_integration.go`: Streams pi events to WebSocket clients
    - `pibridge_test.go`, `event_mapper_test.go`, `e2e_streaming_test.go`, `websocket_integration_test.go`, `performance_test.go`: Tests and benchmarks
  - `api/`: HTTP middleware and WebSocket handling
  - `config/`: Application configuration
  - `database/`: PostgreSQL connection, migrations, and data access
  - `engine/`: Workflow engine orchestrating job execution
  - `initialization/`: Application initialization logic
  - `manager/`: Primitive management (providers, skills, agents, workflows)
  - `primitive/`: Core primitive types and validation
  - `tools/`: Tool registry and implementations (bash, filesystem, http, database)
  - `validation/`: Input validation logic
  - `wasmcompiler/`: WASM module compilation utilities

- **pkg/**: Reusable packages
  - `database/`: Shared database models (Provider, Agent, Workflow, Job, etc.)
  - `job/`: Job queue management, job store, and job execution logic

### PI RPC Integration

The `pirc` package provides the core infrastructure for pi integration:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mule-ai/mule](https://github.com/mule-ai/mule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
