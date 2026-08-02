---
trigger: always_on
description: Henry is a Go library providing generic utility functions and algorithms for slices, maps, and channels. This guide helps AI coding agents work effectively in this codebase.
---

# Agent Guidelines for Henry

Henry is a Go library providing generic utility functions and algorithms for slices, maps, and channels. This guide helps AI coding agents work effectively in this codebase.

## Project Overview

- **Language**: Go 1.18+ (uses generics extensively)
- **Module**: `github.com/modfin/henry`
- **Main Packages**: `slicez`, `mapz`, `chanz`, `mon`, `compare`, `numz`, `pipez`
- **Purpose**: Generic utility functions for common operations on collections

## Build, Test, and Lint Commands

### Running Tests

```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run tests for a specific package
go test ./slicez
go test ./mapz
go test ./chanz

# Run a single test by name
go test -run TestCut ./slicez
go test -run TestEqual ./mapz

# Run tests with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...

# Run tests with race detector
go test -race ./...

# Clear test cache and run tests
go test -count=1 ./...
```

### Building

```bash
# Build all packages
go build ./...

# Install all packages
go get github.com/modfin/henry/...

# Verify dependencies
go mod verify
go mod tidy
```

### Linting

```bash
# Run go vet (built-in static analyzer)
go vet ./...

# Format code
go fmt ./...
gofmt -s -w .

# Check for common issues
go vet ./...
```

## Code Style Guidelines

### Package Organization

- Each major feature area has its own package: `slicez`, `mapz`, `chanz`, etc.
- Support packages: `compare` (comparison utilities), `mon` (monads like Result/Option)
- Test files follow `*_test.go` naming convention
- Tests are in the same package as the code they test

### Imports

- Standard library imports first
- Third-party imports second
- Local package imports last
- Group imports with blank lines between categories

```go
import (
    "errors"
    "fmt"
    
    "github.com/modfin/henry/compare"
    "github.com/modfin/henry/slicez"
)
```

### Generics and Type Parameters

- Use single uppercase letters for simple type parameters: `[A any]`, `[E any]`
- Use descriptive names for constrained types: `[K comparable, V any]`, `[N Ordered]`
- Common type parameter names:
  - `A`, `B`, `C` - generic types in transformations
  - `E` - element type
  - `K` - key type
  - `V` - value type
  - `N` - numeric/ordered type
  - `T` - generic single type

```go
// Good examples from codebase
func Equal[A comparable](s1, s2 []A) bool
func Map[A any, B any](slice []A, mapper func(a A) B) []B
func Keys[K comparable, V any](m map[K]V) []K
```

### Naming Conventions

- **Functions**: PascalCase for exported, camelCase for unexported
- **Variables**: camelCase, descriptive names
- **Constants**: PascalCase or SCREAMING_SNAKE_CASE for package-level
- **Type Parameters**: Single uppercase letters or short descriptive names
- **Receivers**: Short 1-2 letter abbreviations (e.g., `r` for Result)

### Function Design Patterns

#### Immutability
- Most functions return new slices/maps rather than mutating inputs
- Functions that mutate are clearly documented with "Warning mutates" comments
- Clone functions provided when mutation is necessary

```go
// Immutable - returns new slice
func Reverse[A any](slice []A) []A

// Mutable - documented clearly
// Clear will delete all elements from a map
// Warning mutates map
func Clear[K comparable, V any](m map[K]V)
```

#### Function Pairs
- Many functions come in pairs: base version and `By`/`Func` variant
- Base version uses `==` for comparable types
- Variant accepts custom comparison/predicate function

```go
func Equal[A comparable](s1, s2 []A) bool
func EqualBy[E1, E2 any](s1 []E1, s2 []E2, eq func(E1, E2) bool) bool

func Index[E comparable](s []E, needle E) int
func IndexBy[E any](s []E, f func(E) bool) int
```

#### Options Pattern for Channels
- Channel functions use functional options for configuration
- Common options: `OpBuffer`, `OpContext`, `OpDone`

```go
func Map[A any, B any](in <-chan A, mapper func(a A) B, options ...Option) <-chan B
```

### Error Handling

- Return `(value, error)` tuple for operations that can fail
- Use `errors.New()` for simple error messages
- Zero values returned on error
- Result monad available in `mon` package for functional error handling

```go
// Standard Go error handling
func Head[A any](slice []A) (A, error) {
    if len(slice) > 0 {
        return slice[0], nil
    }
    var zero A
    return zero, errors.New("slice does not have any elements")
}

// Result monad pattern (for complex error chains)
func TupleToResult[T any](value T, err error) Result[T] {
    if err != nil {
        return Err[T](err)
    }
    return Ok(value)
}
```

### Comments and Documentation

- All exported functions have GoDoc comments
- Comments start with function name: `// FunctionName does...`
- Include examples in comments when helpful
- Document time complexity for algorithms when relevant
- Clearly mark functions that mutate their arguments

```go
// Clone will copy all keys and values of a map in to a new one
func Clone[K comparable, V any](m map[K]V) map[K]V

// Map will take a chan, in, and executes mapper and put the resulting on to the return chan.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modfin/henry](https://github.com/modfin/henry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
