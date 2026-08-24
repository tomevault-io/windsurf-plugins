---
trigger: always_on
description: This file provides guidance to AI agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Build & Development Commands

```bash
# Build
make build              # Build binary to bin/canvas
make dev                # Build with fmt and vet

# Test
make test               # Run all tests
make test-coverage      # Run tests with coverage report
go test -v ./internal/api/...  # Run specific package tests
go test -v -run TestName ./... # Run single test

# Lint & Format
make fmt                # Format code
make lint               # Run golangci-lint
make vet                # Run go vet

# Install
make install            # Install to /usr/local/bin
make uninstall          # Remove from /usr/local/bin

# Setup
make setup-hooks        # Install git pre-commit hooks
```

## Pre-commit Hook

Run `make setup-hooks` to enable. Runs automatically on each commit:
- `gofmt` - formatting check
- `golangci-lint` - comprehensive linting (if installed)
- `go vet` - static analysis
- `go test -short -race` - quick test pass with race detector

## Architecture

Canvas CLI is a Go CLI for Canvas LMS API, built with Cobra/Viper.

### Project Structure

```
cmd/canvas/     → Entry point (main.go)
commands/       → Cobra command definitions (one file per resource)
  internal/
    options/    → Option structs for commands (eliminates global state)
    logging/    → Structured logging for commands
internal/
  api/          → Canvas API client + service layer (Client, *Service structs)
  auth/         → OAuth 2.0 + PKCE, token storage (keyring/encrypted file)
  config/       → Viper-based configuration management
  cache/        → Response caching with TTL
  batch/        → Concurrent batch operations (worker pool)
  diagnostics/  → canvas doctor checks
  dryrun/       → --dry-run curl rendering
  output/       → Formatters (table, JSON, YAML, CSV)
  progress/     → Progress indicators
  repl/         → Interactive shell
  shellparse/   → Shell-style argument parsing
  telemetry/    → Opt-in usage telemetry
  terminal/     → Terminal capabilities
  update/       → Self-update checks
  webhook/      → Webhook listener
testdata/spec/  → Committed Canvas API spec manifest
tools/          → Code generators (gendocs, speccheck)
.ai/            → Canvas LMS API documentation (gitignored)
```

### Key Patterns

**Service Layer**: Each Canvas resource has a service in `internal/api/`:
```go
type ModulesService struct { client *Client }
func NewModulesService(client *Client) *ModulesService
```

**Command Pattern (NEW)**: Commands should use options structs instead of global flags:
```go
// commands/internal/options/resource.go
type ResourceListOptions struct {
    CourseID int64
    Include  []string
}

func (o *ResourceListOptions) Validate() error {
    return ValidateRequired("course-id", o.CourseID)
}

// commands/resource.go
func newResourceListCmd() *cobra.Command {
    opts := &options.ResourceListOptions{}
    cmd := &cobra.Command{
        Use: "list",
        RunE: func(cmd *cobra.Command, args []string) error {
            if err := opts.Validate(); err != nil {
                return err
            }
            client, _ := getAPIClient()
            return runResourceList(cmd.Context(), client, opts)
        },
    }
    cmd.Flags().Int64Var(&opts.CourseID, "course-id", 0, "Course ID")
    return cmd
}
```

**Structured Logging (NEW)**: Commands should use structured logging:
```go
import "github.com/jjuanrivvera/canvas-cli/commands/internal/logging"

func runCommand(ctx context.Context, client *api.Client, opts *Options) error {
    logger := logging.NewCommandLogger(globalDebugFlag)

    logger.LogCommandStart(ctx, "resource.list", map[string]interface{}{
        "course_id": opts.CourseID,
    })

    // ... perform operation ...

    logger.LogCommandComplete(ctx, "resource.list", len(results))
    return nil
}
```

**API Client**: `internal/api/client.go` provides `HTTPClient` interface with:
- Automatic pagination (`GetAllPages`)
- Adaptive rate limiting based on Canvas quota headers
- Exponential backoff retry

### Testing

Tests use `httptest.NewServer` for mock HTTP servers. Service tests follow pattern:
```go
func TestServiceMethod(t *testing.T) {
    server := httptest.NewServer(...)
    client := &Client{BaseURL: server.URL, ...}
    service := NewXxxService(client)
    // test service methods
}
```

Use `t.Fatal()` (not `t.Error()`) when nil checks would cause subsequent panics.

## Branching & Release

### Branch Model (Simplified Git Flow)

```
main     ──●─────────────────●──────► (tagged releases)
           │                 ↑↓
develop  ──●───●───●───●─────●──────► (integration)
               ↑       ↑
feature/*  ────┘       │
fix/*  ────────────────┘
```

| Branch | Purpose | Merges To |
|--------|---------|-----------|
| `main` | Tagged releases only | - |
| `develop` | Integration (PR target) | `main` on release |
| `feature/*` | New features | `develop` |
| `fix/*` | Bug fixes | `develop` |
| `hotfix/*` | Urgent fixes | `main` AND `develop` |

### When develop syncs with main

1. **After a release**: Merge `main` back to `develop` to capture release commits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jjuanrivvera/canvas-cli](https://github.com/jjuanrivvera/canvas-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
