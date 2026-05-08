---
trigger: always_on
description: This file provides guidance to Claude Code when working with the RTMX Go CLI codebase.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the RTMX Go CLI codebase.

## Overview

This is the Go implementation of the RTMX CLI, providing a single static binary for requirements traceability management. It is a port of the Python CLI (`rtmx-ai/rtmx`).

## Quick Commands

```bash
make build        # Build the binary
make test         # Run tests
make lint         # Run linter (golangci-lint v2 required)
make hooks        # Install pre-commit hooks
make dev          # Build with race detector
make build-all    # Build for all platforms
make parity       # Run parity tests against Python CLI
```

## Local CI Parity

**CRITICAL: Local pre-commit hooks must match remote CI checks.** Run `make hooks` after cloning to install `.githooks/pre-commit` which runs build, test, lint, and vet before each commit. This prevents pushing code that fails CI.

Install golangci-lint v2 if not present:
```bash
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/HEAD/install.sh | sh -s -- -b $(go env GOPATH)/bin
```

## Project Structure

```
rtmx-go/
├── cmd/rtmx/           # Main entry point
├── internal/
│   ├── cmd/            # CLI commands (Cobra)
│   ├── config/         # Configuration management (Viper)
│   ├── database/       # CSV parsing, Requirement model
│   ├── graph/          # Tarjan's SCC, topological sort, critical path
│   ├── output/         # Tables, colors, progress bars
│   ├── adapters/       # GitHub, Jira, MCP integrations
│   └── sync/           # CRDT sync and remotes
├── pkg/rtmx/           # Public API for Go integration
└── testdata/           # Golden files and fixtures
```

## Key Design Decisions

1. **Cobra for CLI** - Standard Go CLI framework with subcommands
2. **Viper for config** - YAML/JSON/ENV config management
3. **No CGO** - Pure Go for static binary distribution
4. **Internal packages** - Implementation details not exported
5. **Golden file tests** - Ensure output parity with Python CLI

## Development Workflow

### Adding New Features or Requirements

**CRITICAL: Fully elaborate the requirement specification and dependency relationships BEFORE commencing any implementation work.** This means:

1. Check `make backlog` for prioritized requirements
2. **Elaborate the requirement specification** in `.rtmx/requirements/<CATEGORY>/`:
   - Write complete acceptance criteria with testable conditions
   - Define all files to create/modify
   - Identify all dependency relationships (blocks/blocked-by)
   - Update `.rtmx/database.csv` with the requirement entry and dependencies
   - Verify no circular dependencies with `rtmx cycles`
3. **Write BDD feature specs** in `features/` when the requirement involves user-facing behavior
4. Write failing tests with proper markers (TDD Red phase)
5. Implement minimal code to pass tests (TDD Green phase)
6. Refactor while keeping tests green
7. Run `make test && make lint`
8. Run `rtmx verify --update` to close the loop
9. Commit with requirement ID in message

### Adding a New Command

1. Create `internal/cmd/<command>.go`
2. Add command to `internal/cmd/root.go` in `init()`
3. Create test file `internal/cmd/<command>_test.go`
4. Add golden files to `testdata/` if needed

### Testing Parity

Every command must produce identical output to the Python CLI:

```bash
# Run parity tests
make parity

# Manual comparison
python -m rtmx status > /tmp/py.out
./bin/rtmx status > /tmp/go.out
diff /tmp/py.out /tmp/go.out
```

### Build & Release

```bash
# Local snapshot build
make snapshot

# Check release config
make release-check

# Release (triggered by tag)
git tag v0.1.0
git push origin v0.1.0
```

## Version Management

Version, commit, and date are injected via ldflags:

```go
// internal/cmd/root.go
var (
    Version = "dev"
    Commit  = "none"
    Date    = "unknown"
)
```

Build with:
```bash
go build -ldflags "-X github.com/rtmx-ai/rtmx/internal/cmd.Version=v0.1.0 ..."
```

## Compatibility Requirements

- **Config files**: Must read rtmx.yaml/.rtmx/config.yaml identically to Python
- **CSV format**: Must read/write rtm_database.csv identically to Python
- **Output format**: Tables, colors, progress bars must match Python
- **Exit codes**: Must match Python exit codes
- **JSON output**: Must match Python JSON schema exactly

## Common Patterns

### Error Handling

```go
func runCommand(cmd *cobra.Command, args []string) error {
    config, err := loadConfig()
    if err != nil {
        return fmt.Errorf("failed to load config: %w", err)
    }
    // ...
}
```

### Color Output

```go
if !noColor && isTerminal() {
    output = colorize(output, "green")
}
```

### Progress Display

```go
bar := output.NewProgressBar(total)
for _, item := range items {
    process(item)
    bar.Increment()
}
bar.Finish()
```

## Dependencies

- `github.com/spf13/cobra` - CLI framework
- `github.com/spf13/viper` - Configuration
- `gopkg.in/yaml.v3` - YAML parsing

Minimize dependencies to keep binary size small (<15MB target).

## Testability Requirements

### Coverage Targets

| Package | Target | Enforcement |
|---------|--------|-------------|
| internal/adapters | 100% | Coveralls + CI gate |
| internal/cmd | 100% | Coveralls + CI gate |
| internal/database | 90% | Coveralls |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rtmx-ai/rtmx-go](https://github.com/rtmx-ai/rtmx-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
