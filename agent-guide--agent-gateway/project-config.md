---
trigger: always_on
description: This repository builds a custom Caddy binary that acts as an AI gateway for LLM, MCP, and ACP traffic.
---

# AGENTS.md

## Purpose

This repository builds a custom Caddy binary that acts as an AI gateway for LLM, MCP, and ACP traffic.
The current primary LLM path is:

1. `agent_gateway` app loads providers, routes, virtual keys, credentials, and CLI auth state
2. `agent_route_dispatcher` matches an incoming HTTP request to a route
3. the route's `protocol` selects the protocol adapter (`openai` or `anthropic`)
4. the gateway validates the VirtualKey
5. in logical-model routes, the model catalog resolves the logical model to one concrete `(provider_id, upstream_model)` binding
6. the selected provider executes `Generate` or `Stream`

MCP is also active now through `agent_route_dispatcher` with MCP enabled, `pkg/gateway/mcproute`, `pkg/mcp/service`, and MCP Admin APIs. ACP is being implemented natively through `pkg/acp`, `pkg/gateway/acproute`, dispatcher turn handling, and ACP Admin APIs. Memory, agent, and metrics areas still exist as earlier-stage subsystems.

## Change Policy

- by default, changes in this repository do not preserve backward compatibility
- do not keep legacy aliases, deprecated field names, old route shapes, old module IDs, old CLI flags, or old API-visible IDs unless the change request explicitly requires compatibility
- when renaming or reshaping behavior, update the code, tests, `README.md`, `docs/architecture/architecture-overview.md`, `Caddyfile.example`, and this file to describe only the current behavior unless compatibility is explicitly required

## Build & Run

```bash
# Build the main gateway binary, standalone daemon, and management CLI
make build

# Or build only the gateway binary
go build -o agw ./cmd/agw

# Or build only the standalone daemon
go build -o agwd ./cmd/agwd

# Or build only the management CLI
go build -o agwctl ./cmd/agwctl

# Run with a Caddyfile
./agw run --config ./Caddyfile.example

# Format
go fmt ./...

# Static analysis
go vet ./...

# Tests
go test ./...
go test ./path/to/package -run TestName -v
```

Notes:

- `make build` builds `agw` from `cmd/agw/main.go`, `agwd` from `cmd/agwd/main.go`, and `agwctl` from `cmd/agwctl/main.go`.
- The resulting binary is a standard Caddy binary with custom modules compiled in, so normal Caddy subcommands such as `run`, `reload`, `validate`, and `hash-password` work.

## Core Modules

### Caddy app

- Module ID: `agent_gateway`
- Package: `caddy/gateway/`
- Main entry: `caddy/gateway/app.go`

Responsibilities:

- initialize the config store
- load static providers from the Caddyfile
- create the shared credential manager and CLI auth refresher
- create the runtime `AgentGateway`

### HTTP middleware

- Module ID: `http.handlers.agent_route_dispatcher`
- Package: `caddy/dispatcher/`
- Main entry: `caddy/dispatcher/dispatcher.go`

Responsibilities:

- resolve the matching `AgentRoute`
- select the route's `protocol`
- rewrite the request path by removing the route `path_prefix`
- validate the VirtualKey
- prepare the provider request payload
- resolve the logical model or direct provider target
- rewrite the provider-facing request model when logical-model routing is used
- invoke the selected LLM protocol handler
- when `mcp` is configured, resolve `MCPRoute` requests, parse MCP JSON-RPC, and invoke `pkg/mcp/service`
- track in-flight MCP requests and progress through the shared runtime registry
- when `acp` is configured, resolve `ACPRoute` requests, parse the gateway ACP turn request, and invoke `pkg/acp/runtime`

### Protocol handler modules

- Module ID: `agent_route_dispatcher.llm_apis.openai`
  - Runtime package: `pkg/dispatcher/llmapi/openai/`
  - Caddy adapter: `caddy/dispatcher/llmapi/openai/`
- Module ID: `agent_route_dispatcher.llm_apis.anthropic`
  - Runtime package: `pkg/dispatcher/llmapi/anthropic/`
  - Caddy adapter: `caddy/dispatcher/llmapi/anthropic/`
- Module ID: `agent_route_dispatcher.llm_apis.cc`
  - Runtime package: `pkg/dispatcher/llmapi/cc/`
  - Caddy adapter: `caddy/dispatcher/llmapi/cc/`

Responsibilities:

- parse wire-format requests
- convert HTTP payloads into `provider.ChatRequest`
- convert provider responses back to protocol-specific JSON or SSE

The `cc` handler is the Claude Code CLI-compatible Anthropic Messages profile. Keep Claude Code CLI-specific protocol shims in this handler rather than in generic providers.

These modules are not standalone `http.handlers.*` modules. They are loaded by `agent_route_dispatcher`.

### Admin API

- Module ID: `http.handlers.agent_gateway_admin`
- Package: `caddy/admin/`

Responsibilities:

- session login with `POST /admin/auth/login`
- CRUD for providers, routes, virtual keys, and credentials
- CRUD for `mcp_services` and MCP routes
- CRUD for `acp_services` and ACP routes
- MCP discovery, execution, and dispatcher runtime inspection
- list startup-enabled provider types and LLM API handler types
- configure and trigger CLI auth authenticators
- start CLI auth logins bound to one `provider_id` and optional credential scope
- expose stubbed memory, agent, and metrics endpoints

## Key Packages

### `pkg/gateway/`

Important files:

- `agentgateway.go`: runtime route, VirtualKey, and provider resolution
- `providerresolver.go`: static and dynamic provider resolution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-guide/agent-gateway](https://github.com/agent-guide/agent-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
