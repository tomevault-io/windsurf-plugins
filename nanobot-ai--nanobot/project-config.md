---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nanobot is a standalone MCP (Model Context Protocol) host that enables building agents with MCP and MCP-UI. Unlike built-in MCP hosts in applications like VSCode, Claude, or ChatGPT, Nanobot is designed to be an open-source, deployable solution that combines MCP servers with LLMs to create agent experiences through various interfaces (chat, voice, SMS, etc.). The project is written in Go (backend) and Svelte 5 + TypeScript (frontend UI).

**Technology Stack:**
- Backend: Go 1.26.0 with GORM (SQLite, MySQL, PostgreSQL), goja (JavaScript runtime for hooks)
- Frontend: Svelte 5, SvelteKit (static adapter), TypeScript, TailwindCSS 4, DaisyUI
- Package Manager: pnpm (for frontend dependencies)

## Build and Development Commands

### Backend (Go)

```bash
# Build the nanobot binary (automatically builds UI via go generate)
make

# Run nanobot with a configuration file
./bin/nanobot run ./nanobot.yaml

# Run Go tests
go test ./...

# Run a specific test
go test ./pkg/agents -run TestName

# Generate code (builds UI and runs Go code generation)
# NOTE: If building manually with `go build`, run this first to ensure UI is embedded
go generate ./...

# Format Go code
gofmt -w .
```

### Frontend (UI)

The UI is a SvelteKit application located in the `./ui` directory. This project uses **pnpm** as the package manager.

```bash
cd ui

# Install dependencies (if needed)
pnpm install

# Start development server (runs on port 5173)
pnpm run dev

# Build for production
pnpm run build

# Lint and format
pnpm run lint
pnpm run format

# Type checking
pnpm run check
```

### Development Workflow for UI

When working on the UI, Nanobot automatically forwards requests to the development server:

1. Remove old build: `rm -rf ./ui/dist`
2. Rebuild backend: `make`
3. Start UI dev server: `cd ui && pnpm run dev`
4. The UI runs on port 5173, while Nanobot backend runs on port 8080 and proxies UI requests

## Architecture Overview

### Backend Architecture

**Core Components:**

- **Runtime (`pkg/runtime/`)** - Main orchestration layer that initializes the system. Creates and wires together the LLM client, tool service, agents, and sampling components. Manages the overall execution environment.

- **Agents (`pkg/agents/`)** - Agent execution engine that handles tool mapping, request population, and agent interactions. Responsible for running agents backed by LLMs with access to tools.

- **Tools Service (`pkg/tools/`)** - Central registry for tools and agents. Handles tool discovery, mapping, and execution delegation. Manages MCP server connections.

- **MCP Layer (`pkg/mcp/`)** - MCP protocol implementation including sessions, clients, servers, and wire protocols. Handles both stdio and HTTP transports. Key types:
  - `Session` - Manages MCP session lifecycle and message routing
  - `Client` - MCP client implementation for connecting to servers
  - `Wire` - Protocol transport abstraction (stdio/HTTP)

- **LLM Integration (`pkg/llm/`)** - Abstraction over different LLM providers (OpenAI, Anthropic). Routes requests to appropriate providers based on model names. Handles both completion and response APIs.

- **Session Management (`pkg/session/`, `pkg/sessiondata/`)** - Manages user sessions, conversation state, and session-scoped data. Handles agent context, tool mappings, and resource mappings within sessions. Supports parent-child session relationships and database-backed OAuth token storage.

- **Server Layer (`pkg/server/`)** - HTTP server handling MCP protocol over HTTP. Routes requests for initialize, tools/list, tools/call, prompts/*, resources/*, etc. Manages session creation and request routing.

- **Built-in MCP Servers (`pkg/servers/`)** - Nanobot includes several built-in MCP servers:
  - `agent/` - Exposes individual agents as MCP servers with chat capabilities
  - `capabilities/` - Session initialization and capability management (workspace setup)
  - `meta/` - Metadata and introspection tools (list_chats, update_chat, list_agents)
  - `resources/` - Database-backed resource management (create_resource, delete_resource) with automatic mimetype detection
  - `workspace/` - Workspace and session management (create/update/delete workspaces, session reading)

- **Configuration (`pkg/config/`)** - YAML-based configuration loading and validation. Supports profiles, extends (inheritance), and environment variables. See `pkg/config/schema.yaml` for the complete schema.

**Key Architectural Patterns:**

- **Tool Mappings** - Tools from MCP servers are mapped to agent-accessible tools. The `BuildToolMappings` method creates this mapping by resolving tool references from agents and MCP servers.

- **Hooks** - Lifecycle hooks for agents and MCP servers (config, request, response). Hooks are TypeScript/JavaScript functions that can modify configuration and messages. See `hooks.ts` for type definitions.

- **Sandboxing** - MCP servers can run in Docker containers for isolation. The `pkg/mcp/sandbox/` handles containerization and port mapping.

### Frontend Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanobot-ai/nanobot](https://github.com/nanobot-ai/nanobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
