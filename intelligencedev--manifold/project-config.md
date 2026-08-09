---
trigger: always_on
description: Source lives in `internal/` (e.g., `internal/agent`, `internal/orchestrator`); keep new packages focused on one concern and avoid import cycles. CLI/Server entrypoints sit under `cmd/` with binaries for the agent, HTTP server (`manifold`, implemented in `cmd/agentd`), and `embedctl` (if present). Docs reside in `docs/`, assets in `assets/`, and deployment scaffolding in `docker/`, `configs/`, and top-level `example.env`. Co-locate tests with their packages and share fakes through `internal/testh
---

# Go/manifold

## Coding Conventions

## Project Structure & Module Organization

Source lives in `internal/` (e.g., `internal/agent`, `internal/orchestrator`); keep new packages focused on one concern and avoid import cycles. CLI/Server entrypoints sit under `cmd/` with binaries for the agent, HTTP server (`manifold`, implemented in `cmd/agentd`), and `embedctl` (if present). Docs reside in `docs/`, assets in `assets/`, and deployment scaffolding in `docker/`, `configs/`, and top-level `example.env`. Co-locate tests with their packages and share fakes through `internal/testhelpers`.

### Package Organization

* Keep packages small and focused on a single responsibility.
* Avoid cyclical dependencies; extract interfaces when necessary.

### Dependency Injection

* Promote testability via interface‑driven design.
* Use constructor functions (e.g., `NewService(...)`) to inject dependencies.

### Concurrency

Follows Go's concurrency mantra: "Don't communicate by sharing memory; share memory by communicating." The channel becomes the synchronization point.

## Essential Go CLI Tools

The following Go command-line tools are essential for development, testing, and maintenance in this project. Use them as described to ensure code quality and consistency:

| Tool         | Purpose                                                                                                 |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| `go build`   | Compiles packages and their dependencies into an executable.                                            |
| `go run`     | Compiles and runs the specified Go program.                                                             |
| `go fmt`     | Formats Go source code according to the language's style guidelines.                                    |
| `gofmt`      | Standalone formatter; also available as an executable.                                                  |
| `go test`    | Runs tests and benchmarks. Use `-coverprofile` with `go tool cover` to analyze test coverage.           |
| `go vet`     | Examines Go source code and reports suspicious constructs that could be bugs.                           |
| `go doc`     | Extracts and generates documentation for Go packages.                                                   |
| `go get`     | Adds, updates, or removes dependencies in the `go.mod` file.                                            |
| `go mod`     | Provides access to module operations (e.g., `go mod tidy` to clean up dependencies).                    |
| `go tool`    | Runs the specified Go tool (see below for examples).                                                    |
| `cgo`        | Enables the creation of Go packages that call C code.                                                   |
| `pprof`      | For profiling Go programs.                                                                              |
| `fix`        | Rewrites Go programs that use old language and library features.                                        |

> **Tip:** Refer to this table whenever you need to build, test, format, or analyze Go code in this project.

## Coding Style & Naming Conventions
Target Go 1.26.3 and keep files `gofmt` clean with tabs. Maintain import order with `goimports` and keep `golangci-lint run` (via `make lint`) silent. Name packages after their capability, keep filenames lowercase, and export concise CamelCase APIs. Prefer constructor-style functions (e.g., `NewService`) for dependency injection.

### Naming Conventions

- Identifiers: use `camelCase` for unexported names and `PascalCase` for exported names; avoid `snake_case` and `SCREAMING_SNAKE_CASE`.
- Acronyms and initialisms: keep casing consistent (`APIKey`, `userID`, `HTTPClient`); avoid mixed forms such as `ApiKey` or `userId`.
- Letters: prefer ASCII identifiers; avoid non-ASCII names unless there is a strong reason.
- Conflicts: do not use names that clash with Go builtins or imported standard library package names.
- Type words: usually omit the type from the identifier unless it is needed to distinguish a converted value.
- Exporting: keep identifiers unexported by default; export only when required by package boundaries or external use.
- Length: use short names for very local scope and more descriptive names as scope widens.
- Packages: use lowercase ASCII only; keep names short, descriptive, and usually one word. Multiword package names should be concatenated lowercase with no separator.
- Packages: avoid standard library package names, invisible prefixes (`.` / `_`), special directories (`vendor`, `testdata`, `internal`), and vague catch-all names such as `util` or `helpers`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intelligencedev/manifold](https://github.com/intelligencedev/manifold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
