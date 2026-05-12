---
trigger: always_on
description: This document provides essential context and rules for AI agents working on this Go API project. It covers the Go backend, the development workflow, and project-wide standards.
---

# AGENTS.md - Project-Wide Conventions

This document provides essential context and rules for AI agents working on this Go API project. It covers the Go backend, the development workflow, and project-wide standards.

# Rule: Context-First Navigation

1. **NEOR (Never Read Without a Reason):** Do not read a file larger than 100 lines using `read_file`. Always use `list_symbols_in_file` first to find the relevant section.
2. **Symbol Over Grep:** If searching for a function, struct, or variable, you MUST use `search_definitions` (SCIP) before attempting `grep`. Grep is a fallback for text; SCIP is for logic.
3. **Structural Extraction:** When you need to understand a specific logic block (like a registration handler), use `ast_grep_scan`. Do not ingest the entire file if you only need one method.
4. **Token Preservation:** Before sending a block of code >500 tokens back to the main loop, call `summarize_code_intent` locally to distill the logic into a high-density summary.

## 1. Development Workflow

The development process is managed by standard Go tooling.

* **Primary Command**: To run the application in development mode, execute the following command from the **project root directory**:
  ```bash
  go run ./...
  ```
* **Building for Production**: To create a production-ready executable, run the following command from the **project root directory**:
  ```bash
  go build -o server ./cmd/server
  ```
* **Restarting the MCP Server**: After building a new `server` executable, the running MCP server **must be restarted** for the changes to take effect. Ask the user to restart the server and wait for their confirmation before attempting to use any updated MCP tools.
* **Running Tests**: To run all tests, execute the following command from the **project root directory**:
  ```bash
  go test ./...
  ```

## 2. Go Backend Conventions

Adherence to these conventions is crucial for a clean, maintainable, and idiomatic Go codebase.

* **Idiomatic Go**: Write clear, simple, and idiomatic Go. Follow standard conventions for naming variables, packages, and interfaces. Avoid creating unnecessary abstractions or wrapper functions.
* **Prefer Copying over Abstraction**: When a small amount of utility code is needed in multiple places, prefer copying the code over creating a new shared package. A little duplication is often better than introducing a premature or incorrect abstraction that creates an unnecessary dependency. This follows the Go proverb: "A little copying is better than a little dependency."
* **Variable Naming**: Follow Go's convention for variable naming. Use short, concise variable names (e.g., `i`, `buf`, `req`) for variables with a limited scope and lifespan. As a variable's scope and distance from its declaration grows, use a longer, more descriptive name (e.g., `userCache`, `incomingRequest`). This improves readability by signaling the variable's importance and scope.
* **Code Formatting**: All Go code must be formatted with `go fmt`. It is strongly recommended to configure your editor to run `go fmt` on save.
* **Logging**: Use the `github.com/charmbracelet/log` package for all logging. Do not use the standard `log` or `fmt` packages for application logging.
* **Testing**: Write unit tests for all new functions, especially for business logic and utility packages. Place test files alongside the code they are testing (e.g., `app_test.go`).
* **GoDoc Comments**: All exported (public) types, functions, constants, and variables **must** have `godoc`-compliant comments. The comment block must begin with the name of the element it is describing, followed by a period. This is the standard format recognized by the `go doc` tool and is essential for generating clean, readable documentation. Do not include extraneous words like "godoc" in the first line of the comment.

## 3. Directory Structure

This project follows the principles of the [Standard Go Project Layout](https://github.com/golang-standards/project-layout).

* **/cmd**: Main application entry points. Each subdirectory is a separate executable.
  * **Application Structure**: Within a specific application's directory (e.g., `/cmd/my-app`), **Do not** create an extra `/internal` directory inside an application's subdirectory (e.g., `/cmd/my-app/internal`). The `cmd` directory structure already prevents cross-application imports, making this redundant.
* **/internal**: Use this directory at the **root level** for code that is shared between multiple applications within this project but should not be exported for use by other projects.
* **/pkg**: Public library code that is safe for external use. This is where reusable packages should be placed.
* **/configs**: Configuration file templates or default configurations.
* **/scripts**: Scripts to perform various build, install, analysis, or other operations.

## 4. Dependency Management (Go Modules)

All Go dependencies must be managed using Go Modules. The `go.mod` and `go.sum` files are the source of truth for the project's dependency graph.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hackafterdark/context-sherpa](https://github.com/hackafterdark/context-sherpa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
