---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Slack CLI is a command-line interface for building apps on the Slack Platform. Written in Go, it provides developers with tools to create, run, deploy, and manage Slack apps locally and remotely.

## Development Commands

### Building

```bash
make build              # Build the CLI (includes linting and cleaning)
make build-ci           # Build for CI (skips lint and tests)
./bin/slack --version   # Run the compiled binary
```

### Testing

```bash
make test                                          # Run all unit tests
make test testdir=cmd/auth testname=TestLoginCommand  # Run specific test
make coverage                                      # View test coverage report
```

### Linting

```bash
make lint                      # Run golangci-lint
go tool golangci-lint version  # Verify linter version
```

### Formatting

Always run `gofmt -w` on changed Go files after making edits to ensure proper formatting.

### Other Commands

```bash
make init               # Initialize project (fetch tags, dependencies)
make clean              # Remove build artifacts (bin/, dist/)
slack docgen ./docs/reference  # Generate command documentation
```

## Architecture

### Project Structure

```
cmd/              Commands (user interface layer)
  ├── auth/       Authentication commands
  ├── app/        App management commands
  ├── platform/   Platform commands (deploy, run, activity)
  ├── project/    Project commands (create, init, samples)
  └── root.go     Root command initialization and alias mapping

internal/         Private packages (implementation details)
  ├── api/        Slack API client and HTTP communication
  ├── app/        App manifest and client logic
  ├── auth/       Authentication handling
  ├── config/     Configuration management
  ├── hooks/      Hook system for lifecycle events
  ├── iostreams/  I/O handling (stdin, stdout, stderr)
  ├── shared/     Shared client factory pattern
  └── experiment/ Feature flag system

main.go           Entry point with tracing and context setup
```

### Key Architectural Patterns

**Command Aliases**: Many commands have shortcut aliases defined in `cmd/root.go`'s `AliasMap`:

- `slack login` → `slack auth login`
- `slack deploy` → `slack platform deploy`
- `slack create` → `slack project create`

**Client Factory Pattern**: `internal/shared/clients.go` provides a `ClientFactory` that manages shared clients and configurations across commands:

- `API()` - Slack API client
- `Auth()` - Authentication client
- `AppClient()` - App management client
- `Config` - Configuration state
- `IO` - I/O streams
- `Fs` - File system (afero)

Commands receive the `ClientFactory` and use it to access functionality.

**Context-Based State**: The codebase uses `context.Context` extensively to pass runtime state (session IDs, trace IDs, versions) through the call stack via `internal/slackcontext`.

**Tracing**: OpenTracing (Jaeger) is initialized in `main.go` for observability.

**Hook System**: `internal/hooks/` implements a lifecycle hook system that allows SDK projects to inject custom behavior at key points. The specification exists in `docs/reference/hooks` files.

**Experiment System**: Features can be gated behind experiments defined in `internal/experiment/experiment.go`:

- Add new experiments as constants
- Register in `AllExperiments` slice
- Enable permanently via `EnabledExperiments`
- Use `--experiment` flag to toggle

### Command Structure

Commands follow this pattern:

1. Define in `cmd/<category>/<command>.go`
2. Create a Cobra command with use/short/long descriptions
3. Add flags specific to that command
4. Implement `RunE` function that receives clients
5. Add unit tests in `*_test.go` alongside

Example command structure:

```go
func NewExampleCommand(clients *shared.ClientFactory) *cobra.Command {
    return &cobra.Command{
        Use:   "example",
        Short: "Brief description",
        RunE: func(cmd *cobra.Command, args []string) error {
            ctx := cmd.Context()
            // Command implementation using clients
            return nil
        },
    }
}
```

### Testing Philosophy

- Unit tests live alongside code with `_test.go` suffix
- Mock implementations use `_mock.go` suffix
- Test code uses syntax compatible with the minimum supported Go version (see `go.mod`)
- The codebase uses `testify` for assertions and `testify/mock` for mocking
- Mock the `ClientFactory` and its dependencies for testing
- Always mock file system operations using `afero.Fs` to enable testability

### Test Naming Conventions

Test function names use the format `Test_StructName_FunctionName` for methods on a struct, or `Test_FunctionName` for package-level functions:

```go
func Test_Client_GetAppStatus(t *testing.T) { ... }  // struct method
func Test_getKeyLength(t *testing.T) { ... }         // package-level function
```

### Test Ordering Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slackapi/slack-cli](https://github.com/slackapi/slack-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
