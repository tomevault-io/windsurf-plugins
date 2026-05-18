---
trigger: always_on
description: This Agents.md file provides comprehensive guidance for AI assistants and coding agents (like Claude, Gemini, Cursor, and others) to work with this codebase.
---

# Project Agents.md for Kubernetes MCP Server

This Agents.md file provides comprehensive guidance for AI assistants and coding agents (like Claude, Gemini, Cursor, and others) to work with this codebase.

This repository contains the kubernetes-mcp-server project,
a powerful Go-based Model Context Protocol (MCP) server that provides native Kubernetes and OpenShift cluster management capabilities without external dependencies.
This MCP server enables AI assistants (like Claude, Gemini, Cursor, and others) to interact with Kubernetes clusters using the Model Context Protocol (MCP).

## Project Structure and Repository layout

- Go package layout follows the standard Go conventions:
  - `cmd/kubernetes-mcp-server/` – main application entry point using Cobra CLI framework.
  - `pkg/` – libraries grouped by domain.
    - `api/` - API-related functionality, tool definitions, and toolset interfaces.
    - `config/` – configuration management.
    - `helm/` - Helm chart operations integration.
    - `http/` - HTTP server and authorization middleware.
    - `kubernetes/` - Kubernetes client management, authentication, and access control.
    - `mcp/` - Model Context Protocol (MCP) server implementation with tool registration and STDIO/HTTP support.
    - `output/` - output formatting and rendering.
    - `toolsets/` - Toolset registration and management for MCP tools.
    - `version/` - Version information management.
- `.github/` – GitHub-related configuration (Actions workflows, issue templates...).
- `docs/` – documentation files (see [Documentation](#documentation) section below).
- `npm/` – Node packages that wraps the compiled binaries for distribution through npmjs.com.
- `python/` – Python package providing a script that downloads the correct platform binary from the GitHub releases page and runs it for distribution through pypi.org.
- `Dockerfile` - container image description file to distribute the server as a container image.
- `Makefile` – tasks for building, formatting, linting and testing.

## Feature development

Implement new functionality in the Go sources under `cmd/` and `pkg/`.
The JavaScript (`npm/`) and Python (`python/`) directories only wrap the compiled binary for distribution (npm and PyPI).
Most changes will not require touching them unless the version or packaging needs to be updated.

### Adding new MCP tools

The project uses a toolset-based architecture for organizing MCP tools:

- **Tool definitions** are created in `pkg/api/` using the `ServerTool` struct.
- **Toolsets** group related tools together (e.g., config tools, core Kubernetes tools, Helm tools).
- **Registration** happens in `pkg/toolsets/` where toolsets are registered at initialization.
- Each toolset lives in its own subdirectory under `pkg/toolsets/` (e.g., `pkg/toolsets/config/`, `pkg/toolsets/core/`, `pkg/toolsets/helm/`).

When adding a new tool:
1. Define the tool handler function that implements the tool's logic.
2. Create a `ServerTool` struct with the tool definition and handler.
3. Add the tool to an appropriate toolset (or create a new toolset if needed).
4. Register the toolset in `pkg/toolsets/` if it's a new toolset.

## Building

Use the provided Makefile targets:

```bash
# Format source and build the binary
make build

# Build for all supported platforms
make build-all-platforms
```

`make build` will run `go fmt` and `go mod tidy` before compiling.
The resulting executable is `kubernetes-mcp-server`.

## Running

The README demonstrates running the server via
[`mcp-inspector`](https://modelcontextprotocol.io/docs/tools/inspector):

```bash
make build
npx @modelcontextprotocol/inspector@latest $(pwd)/kubernetes-mcp-server
```

To run the server locally, you can use `npx`, `uvx` or execute the binary directly:

```bash
# Using npx (Node.js package runner)
npx -y kubernetes-mcp-server@latest

# Using uvx (Python package runner)
uvx kubernetes-mcp-server@latest

# Binary execution
./kubernetes-mcp-server
```

This MCP server is designed to run both locally and remotely.

### Local Execution

When running locally, the server connects to a Kubernetes or OpenShift cluster using the kubeconfig file.
It reads the kubeconfig from the `--kubeconfig` flag, the `KUBECONFIG` environment variable, or defaults to `~/.kube/config`.

This means that `npx -y kubernetes-mcp-server@latest` on a workstation will talk to whatever cluster your current kubeconfig points to (e.g. a local Kind cluster).

### Remote Execution

When running remotely, the server can be deployed as a container image in a Kubernetes or OpenShift cluster.
The server can be run as a Deployment, StatefulSet, or any other Kubernetes resource that suits your needs.
The server will automatically use the in-cluster configuration to connect to the Kubernetes API server.

## Tests

Run all Go tests with:

```bash
make test
```

The test suite relies on the `setup-envtest` tooling from `sigs.k8s.io/controller-runtime`.
The first run downloads a Kubernetes `envtest` environment from the internet, so network access is required.
Without it some tests will fail during setup.

### Testing Patterns and Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [containers/kubernetes-mcp-server](https://github.com/containers/kubernetes-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
