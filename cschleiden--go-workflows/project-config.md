---
trigger: always_on
description: Durable workflows library for Go that borrows heavily from Temporal and DTFx. Supports multiple backends (MySQL, Redis, SQLite, in-memory) and provides comprehensive workflow orchestration capabilities.
---

# go-workflows Development Guide for AI Agents

Durable workflows library for Go that borrows heavily from Temporal and DTFx. Supports multiple backends (MySQL, Redis, SQLite, in-memory) and provides comprehensive workflow orchestration capabilities.

This guide provides essential information for AI coding agents working on this project. Always reference this information first before exploring the codebase or running commands.

## Development Environment Setup

### Bootstrap and Build
Execute these commands in sequence to set up a working development environment:

```bash
# 1. Download Go dependencies - Takes ~3.5 minutes. Allow sufficient timeout.
go mod download

# 2. Build all packages - Takes ~40 seconds.
go build -v ./...

# 3. Start development dependencies - Takes ~13 seconds.
docker compose up -d
```

### Testing
```bash
# Run short tests - Takes ~45 seconds.
go test -short -timeout 120s -race -count 1 -v ./...

# Run full test suite - Takes ~2.5 minutes.
go test -timeout 180s -race -count 1 -v ./...

# Install test reporting tool (if needed)
go install github.com/jstemmer/go-junit-report/v2@latest

# Run tests with JUnit output (as used in CI) - Takes ~2.5 minutes.
go test -timeout 120s -race -count 1 -v ./... 2>&1 | go-junit-report -set-exit-code -iocopy -out "report.xml"
```

### Linting

The project uses golangci-lint v2 with a custom analyzer plugin for workflow code validation. There are multiple ways to run the linter:

#### Recommended: Using Makefile (Preferred)
```bash
# Run the full linter suite with custom analyzer - Takes ~12-15 seconds.
make lint
```

This will:
- Check if golangci-lint is installed
- Build the custom analyzer plugin
- Run all configured linters from `.golangci.yml`

#### Manual Setup
If you need to install golangci-lint or run it manually:

```bash
# Install golangci-lint v2 (required for this project)
go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.4.0

# Run the full linter configuration - Takes ~12-15 seconds.
golangci-lint run --timeout=5m
```

**Note:** The project uses golangci-lint v2.4.0 configuration. Version v1.x will not work with the `.golangci.yml` configuration file.

#### Workaround: Basic Linting (When Custom Analyzer Has Issues)
If the custom analyzer has version compatibility issues:

```bash
# Run basic linting without custom analyzer - Takes ~12 seconds.
golangci-lint run --disable-all --enable=gofmt,govet,ineffassign,misspell --timeout=5m
```

#### Available Linters
The `.golangci.yml` configuration enables multiple linters including:
- **Code Quality**: staticcheck, unused, ineffassign, wastedassign
- **Bug Detection**: govet, makezero, prealloc, predeclared
- **Style & Formatting**: gofmt, whitespace, tagalign
- **Testing**: testifylint, tparallel
- **Custom**: goworkflows (workflow-specific validation, currently commented out)

### Sample Applications
```bash
# Simple workflow example with Redis backend (default)
cd samples/simple && go run .

# Simple workflow with SQLite backend
cd samples/simple && go run . -backend sqlite

# Benchmark utility
cd bench && go run .

# Web example with diagnostic UI
cd samples/web && go run .
# Access UI at http://localhost:3000/diag
```

## Validation and Testing

### Core Workflow Functionality
Always test core workflow functionality after making changes:

1. **Simple End-to-End Test:**
   ```bash
   cd samples/simple && go run . -backend sqlite
   ```
   Expected output: "Workflow finished. Result: 59"

2. **Multi-Backend Test:**
   ```bash
   # Test Redis backend (requires docker compose up)
   cd samples/simple && go run .

   # Test SQLite backend (embedded)
   cd samples/simple && go run . -backend sqlite
   ```

3. **Benchmark Validation:**
   ```bash
   cd bench && go run .
   ```
   Expected: Workflow hierarchy execution with metrics output

### Pre-Commit Validation
Run these validation steps before committing changes:

1. **Full Build:** `go build -v ./...` - Must complete without errors
2. **Short Tests:** `go test -short -timeout 120s -race -count 1 -v ./...` - Should pass (1 known non-critical test failure in tester package)
3. **Linting:** `make lint` or `golangci-lint run --timeout=5m` - Should pass with no new violations
4. **Code Formatting:** `make fmt` - Ensure code is properly formatted
5. **Sample Execution:** At least one sample must run successfully

### Manual Testing Scenarios
Execute these scenarios to verify workflow functionality:

1. **Basic Activity Workflow:**
   - Run `samples/simple`
   - Verify workflow executes activities and returns result
   - Check no errors in output

2. **Subworkflow Execution:**
   - Run `bench` utility
   - Verify hierarchical workflow creation and execution
   - Check metrics output shows expected activity and workflow counts

3. **Backend Switching:**
   - Test same workflow with different backends (Redis, SQLite)
   - Verify consistent behavior across backends

## Repository Structure

### Key Directories
- **`/samples/`** - Example applications demonstrating workflow patterns:
  - `simple/` - Basic workflow with activities
  - `web/` - Web UI example with diagnostics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cschleiden/go-workflows](https://github.com/cschleiden/go-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
