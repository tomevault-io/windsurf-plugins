---
trigger: always_on
description: This Agents.md file provides comprehensive guidance for AI assistants and coding agents (like Claude, Gemini, Cursor, and others) to work with this codebase.
---

# Project Agents.md for Podman MCP Server

This Agents.md file provides comprehensive guidance for AI assistants and coding agents (like Claude, Gemini, Cursor, and others) to work with this codebase.

This repository contains the podman-mcp-server project,
a Go-based Model Context Protocol (MCP) server that provides container management capabilities using Podman or Docker.
This MCP server enables AI assistants (like Claude, Gemini, Cursor, and others) to interact with container runtimes using the Model Context Protocol (MCP).

## Project Structure and Repository layout

- Go package layout follows the standard Go conventions:
  - `cmd/podman-mcp-server/` – main application entry point.
  - `pkg/` – libraries grouped by domain.
    - `api/` - SDK-agnostic types for tool definitions (`ServerTool`, `ToolHandlerFunc`, `ToolHandlerParams`).
    - `config/` - Server configuration (`Config` struct, defaults, and override merging).
    - `mcp/` - Model Context Protocol (MCP) server implementation using the official Go SDK, with tool definitions for containers, images, networks, and volumes.
    - `podman/` - Podman/Docker abstraction layer with interface definition, implementation registry, CLI and REST API implementations.
    - `podman-mcp-server/cmd/` - CLI command definition using Cobra framework.
    - `version/` - Version information management.
  - `internal/test/` – shared test utilities (McpSuite, mock Podman API server).
- `.github/` – GitHub-related configuration (Actions workflows, Dependabot).
- `build/` – modular Makefile includes for packaging targets.
  - `node.mk` – NPM packaging targets (npm-copy-binaries, npm-copy-project-files, npm-publish).
  - `python.mk` – Python/PyPI packaging targets (python-publish).
- `npm/` – Node packages that wrap the compiled binaries for distribution through npmjs.com.
- `python/` – Python package providing a script that downloads the correct platform binary from the GitHub releases page and runs it for distribution through pypi.org.
- `Makefile` – core build tasks; includes `build/*.mk` for packaging targets.
- `server.json` – MCP Registry manifest file for publishing to the official registry.

## Feature development

Implement new functionality in the Go sources under `cmd/` and `pkg/`.
The JavaScript (`npm/`) and Python (`python/`) directories only wrap the compiled binary for distribution (npm and PyPI).
Most changes will not require touching them unless the version or packaging needs to be updated.

### Adding new MCP tools

Tools are currently organized by resource type in `pkg/mcp/`:

- `podman_container.go` - Container tools (inspect, list, logs, remove, run, stop)
- `podman_image.go` - Image tools (build, list, pull, push, remove)
- `podman_network.go` - Network tools (list)
- `podman_volume.go` - Volume tools (list)

When adding a new tool:
1. Identify the appropriate resource file (or create a new one).
2. Define the tool using `api.ServerTool` with the SDK-agnostic types from `pkg/api/`.
3. Implement the handler function using the signature `func(ctx context.Context, params api.ToolHandlerParams) (*api.ToolCallResult, error)`.
4. Add the tool to the `initXTools()` function in the resource file (e.g., `initContainerTools()`).
5. If creating a new resource type, register the `initXTools()` function in `mcp.go` within `NewServer()`.
6. Add tests for the new tool.

Example tool definition:

```go
api.ServerTool{
    Tool: api.Tool{
        Name:        "container_list",
        Description: "List all containers",
        Annotations: api.ToolAnnotations{
            Title:        "List Containers",
            ReadOnlyHint: ptr(true),
        },
        InputSchema: api.InputSchema{
            Type:       "object",
            Properties: map[string]api.Property{
                "all": {Type: "boolean", Description: "Show all containers"},
            },
        },
    },
    Handler: func(ctx context.Context, params api.ToolHandlerParams) (*api.ToolCallResult, error) {
        all := params.GetString("all", "false") == "true"
        result, err := params.Podman.ContainerList(ctx, all)
        return api.NewToolCallResult(result, err), nil
    },
}
```

### SDK Architecture

The project uses a layered architecture that decouples tool definitions from the MCP SDK:

- **`pkg/api/`** - SDK-agnostic types (`ServerTool`, `Tool`, `ToolHandlerParams`, `ToolCallResult`)
- **`pkg/mcp/gosdk.go`** - Conversion layer between internal types and the official Go SDK
- **`pkg/mcp/mcp.go`** - Server wiring that registers tools with the go-sdk

This design allows tool definitions to be written without depending on any specific SDK, making it easier to support multiple transports or SDK versions.

### Podman Interface

The `pkg/podman/interface.go` file defines the `Podman` interface that abstracts container runtime operations.
A registry pattern in `pkg/podman/registry.go` enables multiple implementations with auto-detection.

Available implementations:
- **`cli`** (`pkg/podman/podman_cli.go`) - Uses Podman/Docker CLI commands (priority: 50). Available when `podman` or `docker` binary is in PATH.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manusa/podman-mcp-server](https://github.com/manusa/podman-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
