---
trigger: always_on
description: This document outlines coding standards, patterns, and best practices to be followed when developing Go applications and libraries for the `devkit-cli` project. These rules are based on general Go best practices and observed patterns within this
---

# Go Development Guidelines for devkit-cli

This document outlines coding standards, patterns, and best practices to be followed when developing Go applications and libraries for the `devkit-cli` project. These rules are based on general Go best practices and observed patterns within this 
codebase.

## 0. Always..

- Run `make tests` when completing a task to make sure the entire test suite passes
- Run `make lint` to lint the code

## 1. Code Formatting

- **`gofmt`/`goimports`**: All Go code **MUST** be formatted using `gofmt` or `goimports` before committing. This ensures consistent code style across the project. `goimports` is preferred as it also manages import statements.
    - Configure your IDE to format on save.
    - This is likely enforced by pre-commit hooks.

## 2. Naming Conventions

- **Packages**:
    - Package names **SHOULD** be short, concise, and all lowercase.
    - Avoid overly generic names like `util` or `common` unless the package truly contains cross-cutting concerns. If so, sub-packages within `common` (e.g., `common/httputil`) are preferred.
    - The project uses `pkg/common` for shared utilities (e.g., `logger`, context management), which is acceptable.
- **Variables**:
    - Local variables and function parameters **SHOULD** use `camelCase` (e.g., `myVariable`).
    - Exported variables **MUST** use `PascalCase` (e.g., `ExportedVariable`).
- **Functions and Methods**:
    - Function and method names **SHOULD** use `camelCase` for unexported identifiers (e.g., `calculateValue`).
    - Exported functions and methods **MUST** use `PascalCase` (e.g., `CalculateValue`).
- **Interfaces**:
    - Interfaces **SHOULD** be named with the `-er` suffix if they have only one method (e.g., `Reader`, `Writer`).
    - For more complex interfaces, choose a name that describes its purpose (e.g., `DataStore`).
- **Avoid Stutter**: Do not repeat package names in identifiers. For example, in package `logger`, prefer `logger.New()` over `logger.NewLogger()`.
- **Acronyms**: Acronyms like HTTP, ID, URL **SHOULD** be consistently cased (e.g., `serveHTTP`, `userID`, `parseURL`). `PascalCase` for exported acronyms (e.g., `ServeHTTP`, `UserID`, `ParseURL`).

## 3. Packages and Project Structure

- **`cmd/`**: Main application(s). Each subdirectory in `cmd/` is a separate executable.
- **`pkg/`**: Library code that can be used by other applications or projects. Code here should be designed to be reusable.
    - CLI command logic is well-organized under `pkg/commands`.
- **`internal/`**: Private application and library code. This is the ideal place for code that is specific to this project and should not be imported by other projects.
    - The `internal/version` pattern for build-time variable injection is good.
- **Clarity**: Package structure should clearly communicate the purpose and separation of concerns.
- **Circular Dependencies**: Avoid circular dependencies between packages.

## 4. Error Handling

- **Explicit Handling**: Errors **MUST** be handled explicitly. Do not ignore errors using the blank identifier (`_`) unless there is a very specific and documented reason.
- **Return Errors**: Functions that can fail **MUST** return an `error` as their last return value.
- **Error Wrapping**: When an error is propagated up the call stack, it **SHOULD** be wrapped with additional context using `fmt.Errorf("operation X failed: %w", err)`. This preserves the original error and adds a stack of contextual information.
    - Use `errors.Is()` and `errors.As()` from the standard `errors` package to inspect wrapped errors.
- **Error Messages**: Error messages should be lowercase and not end with punctuation, as they are often combined with other context.
- **Top-Level Handling**: In `main()` or top-level HTTP handlers, errors should be logged appropriately, and the program/request should terminate gracefully (e.g., `log.Fatal(err)` in `main.go` is acceptable for CLI startup).

## 5. Comments and Documentation

- **Godoc**: All exported identifiers (variables, constants, functions, types) **MUST** have Godoc comments.
    - Comments should start with the name of the identifier they describe.
    - Provide clear, concise explanations of what the identifier does, its parameters, and return values.
- **Non-Obvious Code**: Add comments to explain complex, non-obvious, or surprising logic.
- **TODOs**: Use `// TODO:` comments to mark areas that need future attention. Include context or a reference if possible.

## 6. Logging

- **Structured Logging**: Use a structured logging library like `zap` (as currently used in `pkg/common/logger/zap_logger.go`).
- **Log Levels**: Use appropriate log levels (e.g., DEBUG, INFO, WARN, ERROR, FATAL).
- **Contextual Information**: Include relevant contextual information in log messages (e.g., request IDs, user IDs) to aid debugging.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Layr-Labs/devkit-cli](https://github.com/Layr-Labs/devkit-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
