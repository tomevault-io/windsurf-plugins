---
trigger: always_on
description: tmq is a complete, standalone command-line tool for TOML files. Like jq for JSON, yq for YAML — but for TOML. Written in Go with focus on completeness, independence, and script-friendliness.
---

# tmq (TOML Query) - Cursor Development Rules

## Project Overview
tmq is a complete, standalone command-line tool for TOML files. Like jq for JSON, yq for YAML — but for TOML. Written in Go with focus on completeness, independence, and script-friendliness.

## Core Principles
- **Independent**: Single binary, no dependencies on jq, yq, or other tools
- **Complete**: Query, set, delete, convert - everything needed for TOML manipulation
- **Script-friendly**: Pipe support, clear exit codes, CI/CD ready
- **Own Identity**: Not jq-compatible; our own syntax and design

## Development Standards

### 1. Code Quality & Clean Code
- Follow Go best practices and official Go style guide
- Use `gofmt` and `goimports` for consistent formatting
- Meaningful variable/function names, clear comments
- Single responsibility principle for functions
- DRY (Don't Repeat Yourself) principle
- Error handling: return errors, don't panic except in CLI main
- Use context for cancellable operations

### 2. Go Standards Compliance
- Go 1.21+ minimum version
- Use Go modules (`go.mod`)
- Follow standard Go project layout
- Use official Go libraries where possible
- Proper package naming (lowercase, no underscores)
- Use interfaces for testability and dependency injection
- Align with [Effective Go](https://go.dev/doc/effective_go) and [Code Review Comments](https://go.dev/wiki/CodeReviewComments) (receiver names, error handling, imports, line length, etc.)
- Run `go vet` and `gofmt` (or `goimports`); fix all reported issues

### 3. TOML Standards Compliance
- Full TOML 1.0.0 specification compliance
- Support for all TOML data types and structures
- Preserve formatting and comments when possible
- Handle edge cases (arrays, nested tables, inline tables)
- Proper encoding/decoding of special characters

### 4. Testing Requirements
- **Beyond Code Coverage**: Focus on quality metrics, not just coverage percentage
- **F.I.R.S.T Principles**: Fast, Isolated, Repeatable, Self-Checking, Timely
- **Fault Detection**: Tests must actually catch bugs and validate behavior
- **Maintainability**: Tests should be easy to understand, modify, and extend
- **Reliability**: Tests produce consistent results across runs
- **Usability**: Tests serve as executable documentation

#### Unit Test Quality Standards
- **100% test coverage** required, but coverage is necessary not sufficient
- Unit tests for every exported function and method
- **Table-driven tests** for multiple scenarios (Go standard pattern)
- **Subtests** using `t.Run()` for hierarchical organization
- Test **edge cases**, **error conditions**, **boundary values**, and **invalid inputs**
- **Explicit error messages**: Use `t.Errorf()` with clear "got/want" format
- **Helper functions**: Mark with `t.Helper()` for clean stack traces
- **Avoid assertion libraries**: Use Go's built-in testing for better error control
- **Test isolation**: No dependencies between tests, no shared state
- **Fast execution**: Individual tests < 100ms, full suite < 10 seconds
- Mock external dependencies using interfaces and test doubles
- **Behavior testing**: Test what code does, not how it does it
- **Realistic test data**: Use meaningful constants, not random data

#### Go-Specific Testing Patterns
- **Test file naming**: `*_test.go` (see §5 for meaningful names)
- **Test function naming**: `TestXxx` with descriptive names
- **Use Go's built-in testing framework** - no external assertion libraries needed
- **Benchmark tests** for performance validation (`func BenchmarkXxx`)
- **Fuzz tests** for edge case discovery (`func FuzzXxx`)
- **Race detection**: Run tests with `-race` flag
- **Test helpers**: Create reusable helper functions for common assertions

#### Test Quality Metrics
- **Code Coverage**: 80-95% (not 100% absolute - some unreachable code exists)
- **Test-to-Code Ratio**: 1:1 to 2:1 (balance between production and test code)
- **Cyclomatic Complexity**: < 10 per test function
- **Execution Time**: Full test suite < 10 seconds for fast feedback
- **Flakiness Rate**: < 1% (tests should be deterministic)
- **Mutation Testing Score**: Tests should catch artificial defects (future goal)

### 5. Test File Naming & Size (Go Best Practices)
- **Meaningful test file names**: Names must describe what is tested.
  - Prefer `foo_test.go` for tests of `foo.go` when tests are small.
  - When splitting: use focused names, e.g. `query_new_test.go` (tests for New), `query_execute_test.go` (tests for Execute), `parser_file_test.go` (file parsing).
  - Avoid generic names like `misc_test.go` or `extra_test.go`.
- **Test file size**: Keep under 500 lines per file
- **Split large test files**: If >500 lines, split into multiple focused files by behavior/API
- **Test function length**: Individual test functions < 50 lines
- **Table-driven tests**: Can be longer if well-structured (< 100 lines)
- **Naming for split files**: `*_test.go`, `*_internal_test.go`, `*_integration_test.go` only when the suffix meaning is clear
- **Reason**: Tests act as documentation; names should make it obvious what each file validates

#### Test Organization & Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [azolfagharj/tmq](https://github.com/azolfagharj/tmq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
