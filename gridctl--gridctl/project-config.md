---
trigger: always_on
description: Validates the stack spec including config schema, transport rules, and workflow definitions. Exit codes: `0` valid, `1` validation error, `2` infrastructure error.
---

# Gridctl Development Guide

## Project Overview

Gridctl is an MCP (Model Context Protocol) orchestration tool — "Containerlab for MCP Infrastructure".

**Architecture:**
- Controller (Go): Reads stack.yaml, manages Docker containers
- Gateway (Go): Protocol bridge that aggregates tools from downstream MCP servers
- UI (React + React Flow): Visualizes stack with real-time status

## Protocol Bridge Architecture

Gridctl's core value is acting as a **Protocol Bridge** between MCP transports:

```
                    ┌─────────────────────┐
                    │    Claude Desktop   │
                    │    (SSE Client)     │
                    └──────────┬──────────┘
                               │ SSE (GET /sse + POST /message)
                               ▼
                    ┌─────────────────────┐
                    │   Gridctl Gateway    │
                    │  (Protocol Bridge)  │
                    └───┬─────────────┬───┘
                        │             │
           Stdio        │             │  HTTP
    (Docker Attach)     ▼             ▼  (POST /mcp)
              ┌─────────────┐   ┌─────────────┐
              │  MCP Srv A  │   │  MCP Srv B  │
              │ (stdio MCP) │   │ (HTTP MCP)  │
              └─────────────┘   └─────────────┘
```

**Southbound (to MCP servers):**
- **Stdio (Container)**: Uses Docker container attach for stdin/stdout communication
- **Stdio (Local Process)**: Spawns local process on host, communicates via stdin/stdout
- **Stdio (SSH)**: Connects to remote host via SSH, communicates via stdin/stdout over the SSH connection
- **HTTP**: Standard HTTP POST to container's /mcp endpoint
- **External URL**: Connects to MCP servers running outside Docker

**Northbound (to clients):**
- **SSE**: Server-Sent Events for persistent connections (Claude Desktop)
- **HTTP POST**: Standard JSON-RPC 2.0 to /mcp endpoint

## Multi-Network Routing

Gridctl runs as a host binary (like Containerlab, Terraform, Docker Compose), enabling cross-network routing:

```
┌─────────────┐     ┌─────────────┐
│  Network A  │     │  Network B  │
│ (MCP Srv 1) │     │ (MCP Srv 2) │
└──────┬──────┘     └──────┬──────┘
       │   Docker Socket   │
       └─────────┬─────────┘
       ┌─────────▼─────────┐
       │   gridctl binary   │
       │  Routes JSON-RPC  │
       │  through memory   │
       └─────────┬─────────┘
       ┌─────────▼─────────┐
       │   localhost:8180  │
       └───────────────────┘
```

Network isolation between MCP servers while routing through the gateway.

## Directory Structure

```
gridctl/
├── cmd/gridctl/           # CLI entry point
│   ├── main.go           # Entry point
│   ├── root.go           # Cobra root command + serve command
│   ├── apply.go          # Start stack + gateway
│   ├── validate.go       # Validate stack YAML (exit 0/1/2, --format json)
│   ├── plan.go           # Diff spec against running state (--yes, --format json)
│   ├── export.go         # Reverse-engineer stack.yaml from running state (-o, --format)
│   ├── destroy.go        # Stop containers
│   ├── status.go         # Show container status
│   ├── info.go           # Show detected container runtime
│   ├── link.go           # Connect LLM clients to gateway
│   ├── unlink.go         # Remove gridctl from LLM clients
│   ├── reload.go         # Hot reload stack configuration
│   ├── skill.go          # Remote skill management (add, update, remove, pin, info, validate, try)
│   ├── vault.go          # Vault secret management commands
│   ├── pins.go           # Schema pin management commands
│   ├── traces.go         # Distributed traces CLI command (table, waterfall, follow)
│   ├── test.go           # Skill acceptance criteria runner (exit 0/1/2)
│   ├── activate.go       # Skill activation with acceptance criteria enforcement
│   ├── version.go        # Version command
│   ├── upgrade.go        # In-place upgrade to the latest release (sha256-verified)
│   ├── help.go           # Custom help template
│   ├── embed.go          # Embedded web assets
│   └── embed_stub.go     # Build stub for embed
├── internal/
│   ├── server/           # Legacy HTTP server (SPA only)
│   └── api/              # API server (MCP + REST + Registry)
│       ├── api.go        # Server setup and route registration
│       ├── auth.go       # Gateway authentication middleware
│       ├── registry.go   # Registry CRUD endpoints
│       ├── vault.go      # Vault REST API endpoints
│       ├── pins.go       # Schema pins REST API endpoints
│       └── stack.go      # Stack library endpoints (list, save, initialize)
├── pkg/
│   ├── config/           # Stack YAML parsing
│   │   ├── types.go      # Stack, Agent, Resource structs
│   │   ├── loader.go     # LoadStack() function
│   │   ├── expand.go     # Variable expansion (env + vault)
│   │   └── validate.go   # Validation rules
│   ├── dockerclient/     # Docker client interface
│   │   └── interface.go  # Interface definition for mocking
│   ├── logging/          # Logging utilities
│   │   ├── discard.go    # Discard logger
│   │   ├── buffer.go     # In-memory circular log buffer for API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gridctl/gridctl](https://github.com/gridctl/gridctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
