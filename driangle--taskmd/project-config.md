---
trigger: always_on
description: This document provides guidelines and conventions for developing the taskmd project. These instructions are designed to help maintain code quality, consistency, and reliability across the codebase.
---

# Development Guidelines for taskmd

This document provides guidelines and conventions for developing the taskmd project. These instructions are designed to help maintain code quality, consistency, and reliability across the codebase.

## Prerequisites

Before developing, ensure you have the following tools installed:

- **Go** (1.22+): [https://go.dev/dl/](https://go.dev/dl/)
- **pnpm**: `npm install -g pnpm` (for web frontend)
- **golangci-lint**: Required for `make lint` and `make lint-fix`
  - macOS: `brew install golangci-lint`
  - go install: `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest`
  - Other: See [golangci-lint install docs](https://golangci-lint.run/welcome/install/)
- **Git hooks**: Run `git config core.hooksPath .githooks` to enable project git hooks (see [Git Hooks Setup](#git-hooks-setup))

## Testing Requirements

### CLI Testing Policy

**IMPORTANT: All new CLI features MUST include comprehensive tests.**

When implementing new CLI commands or features:

1. **Create test files** in the same package with `_test.go` suffix
   - Example: `internal/cli/graph.go` → `internal/cli/graph_test.go`

2. **Required test coverage** includes:
   - ✅ **Happy path tests** - Verify the feature works correctly with valid inputs
   - ✅ **Format tests** - Test all output formats (JSON, YAML, ASCII, etc.)
   - ✅ **Flag tests** - Test all command-line flags and their combinations
   - ✅ **Error handling** - Test invalid inputs and edge cases
   - ✅ **Integration tests** - Test with real temporary files when applicable

3. **Test naming convention**:
   ```go
   func TestCommandName_FeatureDescription(t *testing.T)
   ```
   Example: `TestGraphCommand_ExcludeStatus_BugFix`

4. **Test structure**:
   ```go
   func TestMyFeature(t *testing.T) {
       // Setup
       tmpDir := createTestFiles(t)

       // Reset flags to known state
       flagVar = defaultValue

       // Execute
       err := runCommand(cmd, args)

       // Verify
       if err != nil {
           t.Fatalf("unexpected error: %v", err)
       }
       // Add specific assertions
   }
   ```

5. **Use test helpers**:
   - Create helper functions for common setup (e.g., `createTestTaskFiles`)
   - Use `t.TempDir()` for temporary directories
   - Use `t.Helper()` in helper functions

6. **Examples of good test coverage**:
   - See `internal/cli/graph_test.go` for a comprehensive example
   - See `internal/graph/graph_test.go` for package-level tests

### Running Tests

```bash
# Run unit/integration tests
cd apps/cli && go test ./...

# Run specific test
go test ./internal/cli -run TestGraphCommand

# Run with verbose output
go test -v ./internal/cli -run TestGraphCommand

# Run with coverage
go test -cover ./...
```

### Running E2E Tests

E2E tests build the real binary and invoke it as a subprocess, testing the full CLI including config loading, argument parsing, and output formatting.

```bash
# Run all e2e tests
cd apps/cli && make e2e

# Run a specific e2e test group
go test -tags e2e -count=1 -run TestConfig ./internal/e2e/...

# Run with verbose output
go test -tags e2e -count=1 -v ./internal/e2e/...
```

**Note:** `make test` does **not** include e2e tests. Run `make e2e` separately.

### Test Coverage Goals

- **CLI commands**: Minimum 80% coverage
- **Core packages** (graph, parser, validator): Minimum 90% coverage
- **Critical paths**: 100% coverage

## Code Quality Standards

### Linting

The project uses `golangci-lint` for code quality checks, following Go conventions:

```bash
# Run linter (from apps/cli directory)
make lint

# Or run directly
golangci-lint run

# Auto-fix issues where possible
make lint-fix

# Run go mod tidy
make tidy

# Run all checks (test, lint, vet)
make check
```

**Key quality metrics enforced**:
- **Function length**: Max 60 lines per function (excluding comments)
  - Promotes single responsibility principle
  - Improves testability and readability
- **Cyclomatic complexity**: Max 15 complexity per function
  - Catches overly complex branching logic
- **Cognitive complexity**: Max 20 cognitive complexity
  - Measures how hard code is to understand
- **Code quality**: errcheck, staticcheck, govet, unused code detection
- **Formatting**: gofmt, goimports

**Note on file length**: Unlike some languages, Go doesn't enforce file-length limits. Instead, the focus is on keeping functions small and focused. If functions stay under 60 lines, files naturally remain manageable.

**Common lint rules**:
- Use `any` instead of `interface{}`
- No unused imports or variables
- Proper error handling (don't ignore errors)
- Consistent naming conventions
- No magic numbers - use named constants

### Go Conventions

1. **Error handling**:
   ```go
   // Good
   if err != nil {
       return fmt.Errorf("operation failed: %w", err)
   }

   // Bad - don't ignore errors
   _ = someOperation()
   ```

2. **Use context for cleanup**:
   ```go
   func TestSomething(t *testing.T) {
       tmpDir := t.TempDir() // Auto-cleanup
       // ... test code
   }
   ```

3. **Consistent formatting**:
   - Use `gofmt` (automatically done by most editors)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [driangle/taskmd](https://github.com/driangle/taskmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
