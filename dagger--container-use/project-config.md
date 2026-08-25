---
trigger: always_on
description: Container-use provides **isolated, containerized development environments** for AI agents by combining:
---

# Agent Guidelines for Container-Use

## Core Architecture

Container-use provides **isolated, containerized development environments** for AI agents by combining:
- **Git branches** for each environment (e.g., `adjective-animal` names like `fluffy-lion`)
- **Dagger containers** with code mounted at `/workdir`
- **Git notes** (`container-use` and `container-use-state` refs) for operation logging and state persistence
- **Worktrees** in `~/.config/container-use/worktrees/` for filesystem representation

**Key Principle**: All operations MUST go through environment tools - NEVER use git commands directly in environments.

## Critical Agent Rules (MUST FOLLOW)

1. **Use ONLY environment tools** for ANY and ALL file, code, or shell operations - NO EXCEPTIONS
2. **DO NOT install or use git CLI** with environment_run_cmd tool - environment tools handle git integration automatically
3. **ALWAYS inform user** how to view your work: `cu log <env-id>` and `cu checkout <env-id>`
4. **Failure to inform user** makes work inaccessible to others

## Build, Test, and Lint Commands

### Build
```bash
# Build binary
go build -o container-use ./cmd/container-use

# Using Dagger
dagger call build --platform=current export --path ./container-use
```

### Test
```bash
# Run all tests
go test ./...

# Unit tests only (fast, no containers)
go test -short ./...

# Integration tests
go test -count=1 -v ./environment

# Run single test
go test -run TestSpecificFunction ./path/to/package
go test -run TestSpecificFunction -v ./path/to/package

# Run tests in specific package
go test ./repository
go test ./environment
```

### Format and Lint
```bash
# Format code (REQUIRED before committing)
go fmt ./...

# Lint
dagger call lint
```

### Dependencies
```bash
# Install dependencies
go mod download

# Clean up dependencies
go mod tidy
```

## Code Style Guidelines

### Imports
- Order: stdlib imports first, then third-party imports
- Group blank lines between stdlib and third-party
- No unused imports (checked by linter)
- Example:
  ```go
  import (
      "context"
      "fmt"
      "os"

      "dagger.io/dagger"
      "github.com/spf13/cobra"
  )
  ```

### Error Handling
- Always wrap errors with context: `fmt.Errorf("description: %w", err)`
- Never suppress errors with empty catch blocks
- Use descriptive error messages that explain what failed
- Example:
  ```go
  if err != nil {
      return fmt.Errorf("failed to open repository: %w", err)
  }
  ```

### Logging
- Use `log/slog` for structured logging
- Context-aware logging with key-value pairs
- Log at appropriate levels (Info, Warn, Error, Debug)
- Example:
  ```go
  slog.Info("Creating environment", "id", env.ID, "workdir", workdir)
  slog.Error("Command failed", "command", cmd, "error", err)
  ```

### Context Usage
- Accept `context.Context` as first parameter in functions that may block or need cancellation
- Pass context through all call chains
- Use `context.Background()` only when no context is available
- Example:
  ```go
  func (r *Repository) Create(ctx context.Context, dag *dagger.Client, title string) (*Environment, error)
  ```

### Naming Conventions
- **Exported types**: PascalCase (`Environment`, `Repository`)
- **Exported functions/methods**: PascalCase (`New()`, `Load()`, `Create()`)
- **Unexported types/methods**: camelCase (`repository`, `loadState`)
- **Constants**: UPPER_SNAKE_CASE for package-level, camelCase for local
- **Interfaces**: Simple names, often -er suffix when appropriate
- **Interfaces**: `Reader`, `Writer`, `LockManager`

### Struct and Type Conventions
- Use struct fields with JSON tags for serialization
- Omit empty fields with `omitempty` tag
- Keep structs focused on single responsibility
- Example:
  ```go
  type EnvironmentConfig struct {
      Workdir         string   `json:"workdir,omitempty"`
      BaseImage       string   `json:"base_image,omitempty"`
      SetupCommands   []string `json:"setup_commands,omitempty"`
  }
  ```

### Function Conventions
- Constructor functions: `New()` for creating new instances
- Getters: No "Get" prefix preferred (e.g., `repo.Workdir()` not `repo.GetWorkdir()`)
- Setters: `Set()` prefix for methods that modify state
- Boolean returns: Return bool, error (not just bool)
- Example:
  ```go
  func New(ctx context.Context, args NewEnvArgs) (*Environment, error)
  func (env *Environment) Workdir() string
  func (kv *KVList) Set(key, value string)
  ```

### Testing Patterns
- **Test file naming**: `<package>_test.go` (e.g., `repository_test.go`)
- **Test function naming**: `Test<FunctionName>` or `Test<FeatureName>`
- **Table-driven tests**: Use for multiple test cases
- **Integration tests**: In `integration/` subdirectory
- **Test helpers**: Create reusable helpers in `test_helpers.go` or `helpers.go`
- **Setup/teardown**: Use `t.Parallel()` for parallel tests, `t.Cleanup()` for teardown
- **Skip tests**: Use `if testing.Short() { t.Skip("...") }` for slow tests
- **Assertions**: Use `require.NoError()` for setup, `assert.*` for assertions

### Constants
- Package-level constants: UPPER_SNAKE_CASE
- Local constants: camelCase
- Use `const` block for related constants
- Example:
  ```go
  const (
      containerUseRemote = "container-use"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dagger/container-use](https://github.com/dagger/container-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
