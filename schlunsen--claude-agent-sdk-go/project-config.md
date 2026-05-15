---
trigger: always_on
description: Do NOT run `make test` or `go test` locally. Tests should only be run in CI.
---

# Claude Agent SDK for Go - Development Guide

## Important: Do Not Run Tests Locally

Do NOT run `make test` or `go test` locally. Tests should only be run in CI.

## Quick Start

### Prerequisites
- Go 1.24+
- Claude Code CLI installed: `npm install -g @anthropic-ai/claude-code`
- Valid `CLAUDE_API_KEY` environment variable

### Build & Test

```bash
# Format code
make fmt

# Run linters
make lint

# Build the SDK
make build

# Run all tests
make test

# Generate coverage report
make coverage

# Clean build artifacts
make clean

# Build and verify examples
make examples
```

## Project Structure

```
claude-agent-sdk-go/
├── README.md                    # User-facing documentation
├── CLAUDE.md                    # This file - dev guidelines
├── GO_PORT_PLAN.md             # Detailed implementation plan
├── go.mod                       # Module definition
├── Makefile                     # Build targets
├── LICENSE                      # MIT License
├── .gitignore                   # Git ignore rules
├── types/                       # Public type definitions (exported)
│   ├── messages.go              # Message types and content blocks
│   ├── messages_test.go         # Message type tests
│   ├── control.go               # Control protocol types
│   ├── control_test.go          # Control protocol tests
│   ├── options.go               # ClaudeAgentOptions builder
│   ├── errors.go                # Error definitions
│   ├── errors_test.go           # Error tests
│   └── doc.go                   # Package documentation
├── internal/                    # Private packages (not exported)
│   ├── transport/               # Transport abstraction
│   │   ├── transport.go         # Transport interface
│   │   └── subprocess_cli.go    # CLI subprocess implementation
│   ├── message_parser.go        # JSON parsing
│   ├── query.go                 # Control protocol handler
│   └── client.go                # Internal client orchestration
├── client.go                    # Public Client type
├── query.go                     # Public Query function
├── doc.go                       # Package documentation
├── examples/                    # Runnable examples
│   ├── simple_query/main.go
│   ├── interactive_client/main.go
│   ├── with_permissions/main.go
│   └── with_hooks/main.go
└── tests/                       # Test files
    ├── *_test.go
    └── testdata/
```

## Reference Implementation

The Python SDK source is available at `/Users/schlunsen/projects/claude-agent-sdk-python` for reference:
- `src/claude_agent_sdk/` - Main implementation
- `src/claude_agent_sdk/_internal/` - Internal modules
- `tests/` - Test patterns and fixtures

Use this as your reference for behavior, message formats, and edge cases while implementing the Go port.

## Development Workflow

### 1. Before Making Changes

Always work on a feature branch, never directly on `main`:

```bash
# Create and switch to feature branch
git checkout -b phase-1/error-types
git checkout -b phase-2/transport-layer
git checkout -b fix/json-parsing
git checkout -b chore/update-makefile
```

Branch naming:
- Features: `phase-N/description` (e.g., `phase-1/error-types`)
- Fixes: `fix/description` (e.g., `fix/goroutine-leak`)
- Chores: `chore/description` (e.g., `chore/update-deps`)

### 2. Making Changes

- Keep changes focused and atomic
- Follow Go style guidelines (see Coding Standards below)
- Add tests for new functionality
- Reference the Python SDK for behavior
- Implement in phase order (types → transport → parser → protocol → API)

### 3. Verifying Changes via CI

Do NOT run tests locally. Instead, commit and push your changes to trigger the GitHub Actions CI pipeline, which is the source of truth for test verification.

**Why CI-only testing:**
- Ensures consistent test environment across all commits
- Tests against multiple Go versions (1.24, 1.25)
- Prevents environment-specific issues
- Provides authoritative pass/fail signal
- Coverage reports are generated automatically

**Workflow:**
```bash
# Format code locally only if needed
make fmt

# Commit changes
git add .
git commit -m "Your message"

# Push to trigger CI
git push origin your-branch-name

# Wait for GitHub Actions to run
# Monitor at: https://github.com/schlunsen/claude-agent-sdk-go/pull/YOUR_PR
```

**Do not run locally:**
- ~~`make test`~~ - Let CI handle this
- ~~`make lint`~~ - CI runs linting
- ~~`make coverage`~~ - CI generates coverage reports

The CI pipeline will verify all changes and report results on the pull request.

### 4. Committing Changes

Keep commits small and focused - one clear change per commit:

```bash
# Good - multiple small commits
git commit -m "Phase 1: Add error types"
git commit -m "Phase 1: Add error wrapping helpers"
git commit -m "Phase 1: Add unit tests for errors"

# Bad - huge commit mixing concerns
git commit -m "Phase 1: Add error types, message parsing, and transport"
```

Commit message format - reference the phase being worked on:

```
Phase 1: Add error types and helpers

- Implement CLINotFoundError, CLIConnectionError, ProcessError
- Add error wrapping with errors.Is() support
- Add test coverage for error creation
```

```
Phase 2: Implement subprocess transport layer

- Add Transport interface abstraction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schlunsen/claude-agent-sdk-go](https://github.com/schlunsen/claude-agent-sdk-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
