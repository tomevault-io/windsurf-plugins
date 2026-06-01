---
trigger: always_on
description: This document provides guidelines for AI agents working on this codebase.
---

# Agent Development Guidelines

This document provides guidelines for AI agents working on this codebase.

## Running Tests

This project uses [mise](https://mise.jdx.dev/) for running tests with enhanced output formatting via gotestsum.

### Run All Tests

```bash
mise test
```

This runs all tests in the project with race detection enabled and provides clean, organized test output.

### Run Tests for Specific Packages

The `mise test` command accepts the same arguments as `go test`, allowing you to target specific packages or use any `go test` flags:

```bash
# Run tests for a specific package
mise test ./openapi/core

# Run tests matching a pattern
mise test -run TestGetMapKeyNodeOrRoot ./openapi/core

# Run tests with verbose output
mise test -v ./marshaller

# Run tests for multiple packages
mise test ./openapi/core ./marshaller

# Use any go test flags
mise test -race -count=1 ./...
```

### Common Test Commands

```bash
# Run all tests in current directory
mise test .

# Run specific test function
mise test -run TestSecurityRequirement_GetMapKeyNodeOrRoot_Success ./openapi/core

# Run tests with coverage
mise run test-coverage

# Run tests without cache
mise test -count=1 ./...
```

### Why Use Mise for Testing?

- **Enhanced Output**: Uses gotestsum for better formatted, more readable test results
- **Consistent Environment**: Ensures correct Go version and tool versions
- **Race Detection**: Automatically enables race detection to catch concurrency issues
- **Submodule Awareness**: Checks for and warns about uninitialized test submodules

## Pre-Commit CI Check

**Always run `mise ci` before committing changes.** This runs the full CI pipeline locally (format, lint, test, build) and ensures your changes won't break CI.

```bash
mise ci
```

## Git Commit Conventions

**Always use single-line conventional commits.** Do not create multi-line commit messages. Do not add `Co-Authored-By` trailers.

### Commit Message Format

```
<type>: <description>
```

### Common Types

- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `refactor:` - Code refactoring
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks
- `perf:` - Performance improvements

### Examples

#### ✅ Good: Single-line conventional commits

```bash
git commit -m "feat: add prefixEncoding and itemEncoding support for OpenAPI 3.2 multipart media types"
git commit -m "fix: correct validation logic for encoding field mutual exclusivity"
git commit -m "test: add comprehensive tests for multipart encoding validation"
git commit -m "refactor: simplify media type context passing in validation"
```

#### ❌ Bad: Multi-line commits

```bash
git commit -m "feat: implement prefixEncoding and itemEncoding for OpenAPI 3.2

- Add PrefixEncoding and ItemEncoding fields to MediaType
- Implement validation for mutual exclusivity
- Add comprehensive tests"
```

### Why Single-Line Commits?

1. **Simplicity**: Easy to read in git log and GitHub UI
2. **Consistency**: All commits follow the same pattern
3. **Searchability**: Easier to search and filter commits
4. **Tool Compatibility**: Works better with automated tools and scripts

## Multi-Module Dependency Management

This repository uses Go workspaces (`go.work`) with multiple modules. The `cmd/openapi` module depends on the root `github.com/speakeasy-api/openapi` module.

### How Local Development Works

The `go.work` file lists all modules, so during local development the workspace resolves cross-module imports automatically. You do **not** need a `replace` directive in `cmd/openapi/go.mod`.

### When Adding New Packages to the Root Module

If you add new packages to the root module (e.g., `oq/`, `graph/`) that `cmd/openapi` imports, the published module version won't contain them yet. The workspace handles this locally, but `cmd/openapi/go.mod` must reference a version that includes the new packages for CI to pass `mod-check`.

**Do NOT use `replace` directives.** Instead:

1. Push your branch with the new root module packages.
2. From the repo root, update `cmd/openapi` to reference your branch commit:
   ```bash
   GOWORK=off go get -C cmd/openapi github.com/speakeasy-api/openapi@<commit-sha>
   GOWORK=off go mod tidy -C cmd/openapi
   ```
3. Verify with `mise run mod-check`.

This gives `cmd/openapi/go.mod` a pseudo-version (e.g., `v1.19.6-0.20260312183335-395c19cd8edd`) that resolves correctly both locally and in CI. Each subsequent push that changes the root module requires repeating step 2 with the new commit SHA.

## CLI Documentation

When adding or modifying subcommands under `openapi spec`, you **must** update both:

1. `README.md` (root) — the command list and Quick Examples section
2. `cmd/openapi/commands/openapi/README.md` — detailed command documentation with examples, flags, and usage patterns

The command README (`cmd/openapi/commands/openapi/README.md`) serves as the primary reference for each subcommand and should include usage examples, flag tables, and before/after demonstrations where applicable.

## Linter Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [speakeasy-api/openapi](https://github.com/speakeasy-api/openapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
