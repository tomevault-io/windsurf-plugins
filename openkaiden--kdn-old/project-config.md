---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

kdn is a command-line interface for launching and managing AI agents (Claude Code, Goose, Cursor, OpenCode, OpenClaw) with custom configurations. It provides a unified way to start different agents with specific settings including skills, MCP server connections, and LLM integrations.

## Build and Test Commands

All build and test commands are available through the Makefile. Run `make help` to see all available commands.

### Build
```bash
make build
```

### Execute
After building, the `kdn` binary will be created in the current directory:

```bash
# Display help and available commands
./kdn --help

# Execute a specific command
./kdn <command> [flags]
```

### Run Tests
```bash
# Run all tests
make test

# Run tests with coverage report
make test-coverage
```

For more granular testing (specific packages or tests), use Go directly:
```bash
# Run tests in a specific package
go test ./pkg/cmd

# Run a specific test
go test -run TestName ./pkg/cmd
```

### Format Code
```bash
# Format all Go files in the project
make fmt

# Check if code is formatted (without modifying files)
make check-fmt
```

Code should be formatted before committing. Run `make fmt` to ensure consistent style across the codebase.

### Git Hooks

The project includes pre-commit and commit-msg hooks in `.githooks/` that enforce formatting checks (`make check-fmt`) and signed-off commits. Activate them once after cloning:

```bash
make setup-hooks
```

### Integration Tests
```bash
# Run integration tests (requires Podman)
make test-integration
```

### Additional Commands
```bash
# Run go vet
make vet

# Run all CI checks (format check, vet, tests)
make ci-checks

# Clean build artifacts
make clean

# Install binary to GOPATH/bin
make install
```

## Architecture

### Command Structure (Cobra-based)
- Entry point: `cmd/kdn/main.go` → calls `cmd.NewRootCmd().Execute()` and handles errors with `os.Exit(1)`
- Root command: `pkg/cmd/root.go` exports `NewRootCmd()` which creates and configures the root command
- Subcommands: Each command is in `pkg/cmd/<command>.go` with a `New<Command>Cmd()` factory function
- Commands use a factory pattern: each command exports a `New<Command>Cmd()` function that returns `*cobra.Command`
- Command registration: `NewRootCmd()` calls `rootCmd.AddCommand(New<Command>Cmd())` for each subcommand
- No global variables or `init()` functions - all configuration is explicit through factory functions

### Global Flags
Global flags are defined as persistent flags in `pkg/cmd/root.go` and are available to all commands.

#### Accessing the --storage Flag

The `--storage` flag specifies the directory where kdn stores all its files. The default path is computed at runtime using `os.UserHomeDir()` and `filepath.Join()` to ensure cross-platform compatibility (Linux, macOS, Windows). The default is `$HOME/.kdn` with a fallback to `.kdn` in the current directory if the home directory cannot be determined.

**Environment Variable**: The `KDN_STORAGE` environment variable can be used to set the storage directory path. The flag `--storage` will override the environment variable if both are specified.

**Priority order** (highest to lowest):
1. `--storage` flag (if specified)
2. `KDN_STORAGE` environment variable (if set)
3. Default: `$HOME/.kdn`

To access this value in any command:

```go
func NewExampleCmd() *cobra.Command {
    return &cobra.Command{
        Use:   "example",
        Short: "An example command",
        Run: func(cmd *cobra.Command, args []string) {
            storagePath, _ := cmd.Flags().GetString("storage")
            // Use storagePath...
        },
    }
}
```

**Important**: Never hardcode paths with `~` as it's not cross-platform. Always use `os.UserHomeDir()` and `filepath.Join()` for path construction.

### Module Design Pattern

All modules (packages outside of `cmd/`) MUST follow the interface-based design pattern to ensure proper encapsulation, testability, and API safety.

**Required Pattern:**
1. **Public types are interfaces** - All public types must be declared as interfaces
2. **Implementations are unexported** - Concrete struct implementations must be unexported (lowercase names)
3. **Compile-time interface checks** - Add unnamed variable declarations to verify interface implementation at compile time
4. **Factory functions** - Provide `New*()` functions that return the interface type

**Benefits:**
- Prevents direct struct instantiation (compile-time enforcement)
- Forces usage of factory functions for proper validation and initialization
- Enables easy mocking in tests
- Clear API boundaries
- Better encapsulation

**This pattern is MANDATORY for all new modules in `pkg/`.**

### JSON Storage Structure

When designing JSON storage structures for persistent data, use **nested objects with subfields** instead of flat structures with naming conventions.

**Preferred Pattern (nested structure):**
```json
{
  "id": "dc610bffa75f21b5b043f98aff12b157fb16fae6c0ac3139c28f85d6defbe017",
  "paths": {
    "source": "/Users/user/project",
    "configuration": "/Users/user/project/.kaiden"
  }
}
```

**Benefits:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openkaiden/kdn-old](https://github.com/openkaiden/kdn-old) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
