---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go validation library (`go-validator`) that provides comprehensive validation for structs, strings, and other data types. It's optimized for Go 1.19+ and emphasizes proper error handling without panics. The library is designed for production use with web frameworks like Gin, Echo, and Iris.

## Key Architecture

### Core Components

- **Validator (`validator.go`)**: Main validation engine with struct tag-based validation
- **Error Handling (`error.go`)**: Comprehensive error types including `Errors`, `FieldError`, and `UnsupportedTypeError`
- **Types (`types.go`)**: Custom validation function types and thread-safe custom rule mapping
- **Cache (`cache.go`)**: Field metadata structures and validation tag parsing
- **Translator (`translator.go`)**: Internationalization support with language-specific error messages
- **Validation Rules**: Split across multiple files:
  - `validator_string.go` - String validation rules (email, alpha, numeric, etc.)
  - `validator_int.go` - Integer validation rules
  - `validator_float.go` - Float validation rules
  - `validator_unit.go` - Unit-specific validations

### Error Handling Architecture

The library uses a sophisticated error handling system:
- `Errors` type: Collection of multiple validation errors
- `FieldError` type: Detailed field-specific errors with optional `FuncError` for internal errors
- Error chaining support with `Unwrap()` method
- Utility methods: `HasFieldError()`, `GetFieldError()`, `GroupByField()`

### Validation Flow

1. **Entry Point**: `ValidateStruct(s interface{})` delegates to the default `Validator` instance
2. **Field Caching**: `cachedTypefields()` caches field metadata in thread-safe `sync.Map` to reduce reflection overhead
3. **Tag Parsing**: Parses `valid:"rule1,rule2=param"` struct tags into `ValidTag` structs
4. **Rule Dispatch**: Routes validation to the appropriate rule map based on rule type:
   - `RuleMap` - No-parameter rules (e.g., `distinct`)
   - `ParamRuleMap` - Parameter rules (e.g., `between=1|10`, `min=5`)
   - `StringRulesMap` - String-specific pattern rules (e.g., `email`, `alpha`, `uuid`)
   - `CustomTypeRuleMap` - User-defined custom validators (thread-safe)
5. **Error Collection**: All validation errors are accumulated in an `Errors` slice

## Development Commands

### Testing
```bash
go test                    # Run all tests
go test -v                 # Run tests with verbose output
go test -run TestName      # Run specific test
go test -bench=.           # Run benchmarks
go test -bench=. -benchmem # Run benchmarks with memory stats
go test -cover             # Run tests with coverage
go test -race              # Run tests with race detection
```

### Building & Validation
```bash
go build                   # Build the package
go mod tidy                # Clean up dependencies
go vet                     # Run go vet for static analysis
go fmt                     # Format code
gofmt -s -w .              # Format and simplify code
```

### Performance Testing
```bash
go test -bench=BenchmarkErrorHandling           # Test error handling performance
go test -bench=BenchmarkGo119Performance        # Test Go 1.19 optimizations
go test -bench=BenchmarkMemoryAllocation        # Test memory allocation patterns
go test -bench=BenchmarkStringBuilderOptimization # Test string building performance
go test -bench=BenchmarkFuncErrorHandling       # Test FuncError functionality performance
go test -bench=BenchmarkErrorUnwrapping         # Test error unwrapping performance
```

## Usage Patterns

### Basic Struct Validation
```go
type User struct {
    Name  string `valid:"required"`
    Email string `valid:"required,email"`
    Age   int    `valid:"min=18"`
}

err := validator.ValidateStruct(user)
if err != nil {
    errors := err.(validator.Errors)
    // Use utility methods for error handling
    if errors.HasFieldError("Email") {
        fieldError := errors.GetFieldError("Email")
        fmt.Println("Email error:", fieldError.Message)
    }
    
    // Group errors by field for organized display
    groupedErrors := errors.GroupByField()
    for field, errs := range groupedErrors {
        fmt.Printf("Field %s has %d errors\n", field, len(errs))
    }
}
```

### Nested, Embedded, and Recursive Structs

- **Nested structs** (named or pointer) are validated **recursively and
  automatically** — a field no longer needs its own `valid` tag for its inner
  fields to be checked. `time.Time`/`decimal.Decimal` are treated as scalar
  types, not recursed into.
- **Embedded (anonymous) structs** have their promoted fields validated as if
  declared on the parent. A nil embedded pointer is skipped (no panic).
- **Cyclic references** (a node pointing back to an ancestor) terminate via a
  depth guard (`maxValidationDepth`) that returns an error instead of looping
  forever. Validation is linear in the number of fields/elements.

### Fail-Fast vs Collect-All

By default the validator collects every error. Set `FailFast` on a `Validator`
instance to stop at the first field that fails and return immediately (similar to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syssam/go-validator](https://github.com/syssam/go-validator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
