---
trigger: always_on
description: **Clancy** is a robust loop orchestrator written in Go that automates the execution of AI coding agents (like `claude code` or other AI tools). It runs agents in a persistent loop until a specific success criteria (a "safe word" or stop phrase) is met or safety limits are reached. The tool provides automated looping, safety limits (max steps, global timeout), input resolution from files, and cross-platform support (Linux, macOS via PTY, Windows via standard pipes).
---

# Agent Context for Clancy

## Summary

**Clancy** is a robust loop orchestrator written in Go that automates the execution of AI coding agents (like `claude code` or other AI tools). It runs agents in a persistent loop until a specific success criteria (a "safe word" or stop phrase) is met or safety limits are reached. The tool provides automated looping, safety limits (max steps, global timeout), input resolution from files, and cross-platform support (Linux, macOS via PTY, Windows via standard pipes).

## Maintaining this Document

After completing any task, review this file and update it if you made structural changes or discovered patterns worth documenting. Only add information that helps understand how to work with the project. Avoid implementation details, file listings, or trivial changes. This is a general guide, not a changelog.

When updating this document, do so with the context of the entire document in mind; do not simply add new sections at the end, but place them where they make the most sense within the context of the document.

## Tech Stack

- **Language**: Go 1.25
- **Key Libraries**:
  - `github.com/alexflint/go-arg` - Command line argument parsing
  - `github.com/creack/pty` - PTY (pseudo-terminal) support for Unix systems
  - `github.com/matoous/go-nanoid/v2` - ID generation for config files
  - `gopkg.in/yaml.v3` - YAML configuration parsing
  - `github.com/stretchr/testify` - Testing framework and mocking
- **Build Tool**: Task (Taskfile.yml)
- **CI/CD**: GitHub Actions with devcontainers
- **Linter**: golangci-lint

## Operational Commands

### Build & Run
- **Build**: `task build` (Builds to `./dist/clancy`)
- **Run**: `task run` (Builds and executes)
- **Manual Build**: `go build -o ./dist/clancy ./cmd/clancy/.`

### Testing & Quality
- **Test**: `task test` or `go test -v ./...` - Runs all tests with verbose output
- **Lint**: `task lint` or `golangci-lint run` - Runs golangci-lint
- **Format**: `task fmt` or `go fmt ./...` - Formats Go code

### CI/CD
- **Full CI**: `task ci` - Runs format, lint, test, and build in sequence
- **CI in GitHub Actions**: Uses devcontainers with `task ci` command
- **Cross-Platform Build**: `task build-all` - Builds binaries for Linux (amd64/arm64), macOS (amd64/arm64), and Windows (amd64/arm64)
- **Clean Artifacts**: `task clean` - Removes all build artifacts from dist/

## General Instructions

- **Context**: Always review the project structure via `tree -L 2` before making changes. This is a clean architecture Go project with clear separation between entry point (`cmd/`) and business logic (`internal/`).
- **Tools**: Always use Task (`task <taskname>`) for build, test, lint, and format operations. Do not use direct `make` or shell commands.
- **Verification**: After any changes, run `task ci` to ensure format, lint, tests, and build all pass before considering the task complete.
- **Cross-Platform**: Code must work on Linux, macOS, and Windows. Platform-specific implementations are in `internal/runner/` (e.g., `runner_unix.go`, `runner_windows.go`).
- **Interface-Based Design**: Core components use interfaces (e.g., `AgentRunner`) to enable mocking and testing. Follow this pattern for new components.
- **Configuration**: All behavior is driven by YAML configuration files. The template is embedded in `cmd/clancy/template.yaml`.
- **Error Handling**: Use idiomatic Go error handling with wrapped errors (`fmt.Errorf` with `%w`). Return descriptive errors that help users understand what went wrong.
- **Idiomatic Go**:
  - Follow standard Go project layout
  - Use descriptive, self-documenting names
  - Prefer explicit returns over named returns
  - Use `go generate` comments for embedded files (see `//go:embed` in main.go)
  - Keep functions focused and small
- **Testing**: Write tests for all new functionality using `testing` and `testify` for assertions and mocks. Test files should be in the same package as the code they test (e.g., `loop_test.go` alongside `loop.go`).

## Architecture & Organization

### Root Layout
- `go.mod`: Go module definition and dependencies
- `go.sum`: Go dependency checksums
- `Taskfile.yml`: Task runner configuration (build, test, lint, format, cross-platform builds)
- `install.sh`: Installation script for downloading and installing pre-built binaries
- `clancy`: Compiled binary (in git for distribution purposes)
- `clancy*.yaml`: Configuration files (examples and generated configs)
- `README.md`: User documentation and usage guide
- `LICENSE`: MIT license
- `.github/workflows/ci.yaml`: CI/CD pipeline configuration using devcontainers
- `.github/workflows/release.yaml`: Release workflow for publishing releases

### `cmd/clancy/` (Entry Point)
- **Role**: Application entry point and CLI interface
- **Key Files**:
  - `main.go`: Main function, argument parsing, and orchestration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eduardolat/clancy](https://github.com/eduardolat/clancy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
