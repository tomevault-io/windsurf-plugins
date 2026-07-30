---
trigger: always_on
description: Instructions for writing Go code following idiomatic Go practices and community standards
---


# Go Development Instructions

Follow idiomatic Go practices and community standards when writing Go code.
These instructions are based on [Effective Go](https://go.dev/doc/effective_go),
[Go Code Review Comments](https://go.dev/wiki/CodeReviewComments),
[Go Test Code Comments](https://go.dev/wiki/TestComments),
and [Google's Go Style Guide](https://google.github.io/styleguide/go/).

## Go Version Awareness

- Always determine the project's Go version from the `go` directive in `go.mod` before suggesting changes.
- Do **not** propose rewrites that replace modern standard-library features (Go 1.21+) with hand-rolled equivalents.
- Do **not** flag, downgrade, or "polyfill" the following modern features when the `go.mod` permits them:
  - Go 1.20: `errors.Join`.
  - Go 1.21: built-ins `min`, `max`, `clear`; stdlib packages `slices`, `maps`, `cmp`; structured logging `log/slog`.
  - Go 1.22: range-over-integer (`for i := range n`); per-iteration loop variable scoping; `math/rand/v2`; enhanced `http.ServeMux` patterns; `cmp.Or`.
  - Go 1.23: range-over-function iterators (`iter.Seq`, `iter.Seq2`); `unique` package; `slices.Sorted`, `slices.Collect`, etc.
  - Go 1.24: generic type aliases; `omitzero` JSON tag; `weak` package; `testing.B.Loop`; `os.Root`.
  - Go 1.25: `testing/synctest`; `runtime.AddCleanup`; container-aware `GOMAXPROCS`; `encoding/json/v2` (experimental).
- When proposing code that requires a feature newer than `go.mod` allows, call this out explicitly rather than silently using it.
- Prefer the newest idiomatic form available for the project's Go version; only suggest the older pattern when the feature is genuinely unavailable.

## General Instructions

- Write simple, clear, and idiomatic Go code
- Favor clarity and simplicity over cleverness
- Follow the principle of least surprise
- Keep the happy path left-aligned (minimize indentation)
- Return early to reduce nesting
- Make the zero value useful
- Document exported types, functions, methods, and packages
- Use Go modules for dependency management

## Naming Conventions

### Packages

- Use lowercase, single-word package names
- Avoid underscores, hyphens, or mixedCaps
- Choose names that describe what the package provides, not what it contains
- Avoid generic names like `util`, `common`, or `base`
- Package names should be singular, not plural

### Variables and Functions

- Use mixedCaps or MixedCaps (camelCase) rather than underscores
- Keep names short but descriptive
- Use single-letter variables only for very short scopes (like loop indices)
- Exported names start with a capital letter
- Unexported names start with a lowercase letter
- Avoid stuttering (e.g., avoid `http.HTTPServer`, prefer `http.Server`)

### Interfaces

- Name interfaces with -er suffix when possible (e.g., `Reader`, `Writer`, `Formatter`)
- Single-method interfaces should be named after the method (e.g., `Read` → `Reader`)
- Keep interfaces small and focused

### Constants

- Use MixedCaps for exported constants
- Use mixedCaps for unexported constants
- Group related constants using `const` blocks
- Consider using typed constants for better type safety

## Code Style and Formatting

### Formatting

- Always use `gofmt` to format code
- Use `goimports` to manage imports automatically
- Keep line length reasonable (no hard limit, but consider readability)
- Add blank lines to separate logical groups of code

### Comments

- Write comments in complete sentences
- Start sentences with the name of the thing being described
- Package comments should start with "Package [name]"
- Use line comments (`//`) for most comments
- Use block comments (`/* */`) only for files in `testdata`
- Document why, not what, unless the what is complex

### Error Handling

- Check errors immediately after the function call
- Don't ignore errors using `_` unless you have a good reason (document why)
- Wrap errors with context using `fmt.Errorf` with `%w` verb
- Create custom error types when you need to check for specific errors
- Place error returns as the last return value
- Name error variables `err`
- Keep error messages lowercase and don't end with punctuation

## Architecture and Project Structure

### Package Organization

- Follow standard Go project layout conventions
- Use `internal/` for packages that shouldn't be imported by external projects
- Group related functionality into packages
- Avoid circular dependencies

### Dependency Management

- Use Go modules (`go.mod` and `go.sum`)
- Keep dependencies minimal
- Regularly update dependencies for security patches
- Use `go mod tidy` to clean up unused dependencies

## Type Safety and Language Features

### Type Definitions

- Define types to add meaning and type safety
- Use struct tags for JSON, XML, database mappings
- Prefer explicit type conversions
- Use type assertions carefully and check the second return value

### Pointers vs Values

- Use pointers for large structs or when you need to modify the receiver
- Use values for small structs and when immutability is desired
- Be consistent within a type's method set
- Consider the zero value when choosing pointer vs value receivers

### Interfaces and Composition

- Accept interfaces, return concrete types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mgechev/revive](https://github.com/mgechev/revive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
