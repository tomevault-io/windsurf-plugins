---
trigger: always_on
description: **Prerequisites**: Go 1.24+, golangci-lint, Docker (for integration tests), npx (for inspector)
---

# Portainer MCP Development Guide

## Build, Test & Run Commands
**Prerequisites**: Go 1.24+, golangci-lint, Docker (for integration tests), npx (for inspector)

- Build: `make build`
- Unit tests: `make test` (excludes integration tests)
- Run single test: `go test -v ./path/to/package -run TestName`
- Test with coverage: `make test-coverage` (generates `coverage.out`)
- Integration tests: `make test-integration` (requires Docker; spins up Portainer containers)
- Run all tests: `make test-all` (unit + integration)
- Lint: `make lint` (runs `golangci-lint run ./...`)
- Format code: `gofmt -s -w .`
- Run inspector: `make inspector`
- Build for specific platform: `make PLATFORM=<platform> ARCH=<arch> build`
- Clean build artifacts: `make clean`

### CLI Flags
```
dist/portainer-mcp \
  -server <portainer-url> \   # Required: Portainer server URL
  -token <api-token> \        # Required: Portainer API token
  -tools <path>               # Optional: path to tools.yaml (default: tools.yaml)
  -read-only                  # Optional: restrict to read-only tools
  -disable-version-check      # Optional: skip Portainer version validation
```

## Code Style Guidelines
- Use standard Go naming conventions: PascalCase for exported, camelCase for private
- Follow table-driven test pattern with descriptive test cases
- Error handling: return errors with context via `fmt.Errorf("failed to X: %w", err)`
- Imports: group standard library, external packages, and internal packages
- Function comments: document exported functions with Parameters/Returns sections
- Use functional options pattern for configurable clients
- Lint with `make lint` before committing; fix all warnings
- Package structure: cmd/ for entry points, internal/ for implementation, pkg/ for reusable components
- Models belong in pkg/portainer/models, client implementations in pkg/portainer/client
- Tool YAML parsing and parameter extraction in pkg/toolgen
- Kubernetes resource annotation stripping in internal/k8sutil

## Design Documentation
- Design decisions are documented in individual files in `docs/design/` directory
- Follow the naming convention: `YYYYMM-N-short-description.md` where:
  - `YYYYMM` is the year and month
  - `N` is a sequence number for that month
  - Example: `202504-1-embedded-tools-yaml.md`
- Use the standard template structure provided in `docs/design_summary.md`
- Add new decisions to the table in `docs/design_summary.md`
- Review existing decisions before making significant architectural changes

## Client and Model Guidelines

### Two API Implementation Paths

The `PortainerClient` in `pkg/portainer/client/client.go` holds two internal clients (`cli` for the SDK, `rawCli` for raw HTTP) because the SDK doesn't cover the full Portainer API. **When adding new operations, choose the right path:**

#### Path 1: SDK Client (preferred when available)
- Uses `github.com/portainer/client-api-go/v2`
- Covers: environments, tags, teams, users, groups, access groups, edge stacks, settings, Docker/K8s proxy
- Authentication is handled internally by the SDK (token passed at initialization)
- Pattern: call `c.cli.SomeMethod()` → convert SDK models to local models → return
- Example files: `client/environment.go`, `client/tag.go`, `client/stack.go`
- **Unit tests**: mock the `PortainerAPIClient` interface via `MockPortainerAPI` in `mocks_test.go`

#### Path 2: Raw HTTP Client (when SDK has no method)
- Uses direct HTTP via `c.rawCli.apiRequest(method, path, body)`
- Currently used for: local (non-edge) Docker Compose stacks (`/api/stacks/*`)
- Authenticates with `X-API-Key` header (set in `apiRequest` helper)
- You must manually: build the URL path, marshal request bodies, check `resp.StatusCode`, decode JSON responses
- Define request/response structs locally in the same file (e.g., `createLocalStackRequest`)
- Example file: `client/local_stack.go`
- **Unit tests**: use `httptest.NewServer` to stand up a fake HTTP server (no mock interface)

#### Decision Guide for New Features

| Check | Use |
|-------|-----|
| SDK has a method for the endpoint | SDK path (`c.cli`) |
| SDK has no method, Portainer REST API exists | Raw HTTP path (`c.rawCli.apiRequest`) |
| Proxying Docker/K8s APIs through Portainer | SDK path (uses `c.cli.ProxyDockerRequest` / `ProxyKubernetesRequest`) |

**Gotcha**: Both paths share the same `PortainerClient` wrapper and expose methods through the same `PortainerClient` interface in `internal/mcp/server.go`. Callers (MCP handlers) don't know or care which path is used — the wrapper abstracts it.

### Model Structure
1. **Raw Models** (`portainer/client-api-go/v2/pkg/models`)
   - Direct mapping to Portainer API data structures
   - May contain fields not relevant to MCP
   - Prefix variables with `raw` (e.g., `rawSettings`, `rawEndpoint`)

2. **Local Models** (`pkg/portainer/models`)
   - Simplified structures tailored for the MCP application
   - Contain only relevant fields with convenient types
   - Define conversion functions to transform from Raw Models

3. **Raw HTTP Models** (for raw HTTP path only)
   - `RawLocalStack` in `pkg/portainer/models/stack.go` — maps directly to Portainer JSON (PascalCase field tags)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [portainer/portainer-mcp](https://github.com/portainer/portainer-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
