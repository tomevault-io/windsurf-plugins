---
trigger: always_on
description: **go-meridian** is a Go library that provides type-safe timezone handling using Go generics. It solves a fundamental problem: timezone information in `time.Time` is data, not type, and can be lost without the compiler noticing. Meridian makes timezone information immutable by encoding it directly into the type system.
---

# AGENTS.md

## Project Overview

**go-meridian** is a Go library that provides type-safe timezone handling using Go generics. It solves a fundamental problem: timezone information in `time.Time` is data, not type, and can be lost without the compiler noticing. Meridian makes timezone information immutable by encoding it directly into the type system.

**Core Philosophy**: "Make wrong timezone handling impossible to compile."

This is a **distributable library** intended for consumption by other Go projects, not an application. All changes should maintain backwards compatibility and API stability.

## Key Concepts

### 1. Timezones as Types
- `meridian.Time[TZ]` carries timezone information in its type parameter
- `meridian.Time[est.EST]` and `meridian.Time[pst.PST]` are **different types**
- The compiler prevents accidental timezone mixing or loss

### 2. Per-Timezone Packages
- Each timezone lives in its own package: `aest`, `brt`, `cet`, `cst`, `ct`, `et`, `gmt`, `hkt`, `ist`, `jst`, `mt`, `pt`, `sgt`, `utc`, etc.
- Timezone packages provide helper functions: `et.Now()`, `pt.Date(...)`, etc.
- Type aliases enable clean signatures: `utc.Time`, `et.Time`, `pt.Time`
- Package name conveys timezone, type is always `Timezone`

### 3. Explicit Conversions
- Timezone conversions must be explicit: `est.FromMoment(pacificTime)`
- Conversions use the `Moment` interface, supporting both `time.Time` and `meridian.Time[TZ]`
- This makes timezone handling visible in code review
- No silent timezone changes or data loss
- All conversions preserve the moment in time (UTC equality)

### 4. Internal UTC Storage
- All times are stored as UTC internally (`utcTime time.Time`)
- Timezone is applied during operations (display, hour extraction, etc.)
- Database-friendly and eliminates ambiguity

## Project Structure

```
.
├── meridian.go              # Core package: Time[TZ] type and core functions
├── meridian_test.go         # Core package tests
├── example_test.go          # Testable examples (appear in godoc)
├── doc.go                   # Package-level documentation
├── cmd/example/main.go      # Example usage program
├── est/                     # Eastern Time timezone package
│   ├── est.go
│   └── est_test.go
├── pst/                     # Pacific Time timezone package
│   ├── pst.go
│   └── pst_test.go
├── utc/                     # UTC timezone package
│   ├── utc.go
│   └── utc_test.go
├── .golangci.yml            # Linter configuration
└── Makefile                 # Development commands
```

## Development Commands

### Quick Reference
```bash
make test           # Run tests with race detection
make test-coverage  # Generate coverage report (coverage.html)
make lint           # Run golangci-lint
make run-example    # Run the example program
make clean          # Remove build artifacts
make install-tools  # Install development dependencies
```

### Full Commands
```bash
# Testing
go test -v -race ./...                                    # Run all tests
go test -v -race -coverprofile=coverage.out ./...        # With coverage
go tool cover -html=coverage.out                         # View coverage

# Linting
golangci-lint run                                         # Run all linters

# Running examples
go run cmd/example/main.go                                # Run example
```

## Testing Requirements

### Must-Have for All Changes
1. **Tests must pass**: `make test` must succeed (includes race detection)
2. **Coverage should not decrease**: Run `make test-coverage` to verify
3. **All linters must pass**: `make lint` must show no errors
4. **Examples must compile**: Testable examples in `example_test.go` must be valid

### Testing Philosophy
- Write table-driven tests for multiple scenarios
- Test edge cases: DST transitions, leap seconds, year boundaries
- Use `t.Parallel()` for independent tests
- Test both the generic API (`meridian.Time[TZ]`) and timezone-specific helpers

### Race Detection
Always run tests with `-race` flag. This is a library dealing with time, which often involves concurrency. The CI pipeline enforces this.

## Code Style Guidelines

### Enforced by Linters
- **gofmt**: Standard Go formatting
- **goimports**: Proper import ordering and grouping
- **revive**: Naming conventions, exported types must have comments
- **godot**: Comments must end with punctuation
- **errcheck**: All errors must be handled
- **gosec**: Security best practices
- **gocyclo**: Max complexity 15

### Type-Specific Conventions

#### Generic Functions
```go
// Good: Generic function with timezone type parameter
func Now[TZ Timezone]() Time[TZ] {
    return Time[TZ]{utcTime: time.Now().UTC()}
}

// Good: Preserving timezone types through operations
func Add[TZ Timezone](t Time[TZ], d time.Duration) Time[TZ] {
    return Time[TZ]{utcTime: t.utcTime.Add(d)}
}
```

#### Timezone Package Functions
```go
// Good: Timezone-specific helper (in est/est.go)
func Now() meridian.Time[EST] {
    return meridian.Now[EST]()
}

// Good: Timezone-specific constructor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthalp/go-meridian](https://github.com/matthalp/go-meridian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
