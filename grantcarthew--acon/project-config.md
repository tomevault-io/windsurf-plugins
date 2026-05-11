---
trigger: always_on
description: acon (Atlassian Confluence) is a CLI tool for managing Confluence pages and spaces from the terminal. It provides bidirectional Markdown conversion (Markdown ↔ Confluence storage format), enabling a local-first documentation workflow.
---

# AGENTS.md

## Project Overview

acon (Atlassian Confluence) is a CLI tool for managing Confluence pages and spaces from the terminal. It provides bidirectional Markdown conversion (Markdown ↔ Confluence storage format), enabling a local-first documentation workflow.

- Active Project: .ai/projects/p-001-cli-search-command.md
- Design Record: .ai/design/design-records/dr-001-cli-search-c
  ommand.md

Technology Stack:

- Go 1.25.4
- Cobra (CLI framework)
- Goldmark (Markdown parser with GFM extension)
- html-to-Markdown v2 (Confluence storage to Markdown)
- Confluence REST API v2 (primary) and v1 (search endpoint only)

## Setup Commands

```bash
# Install dependencies
go mod download

# Build the binary
go build -o acon

# Run directly without building
go run main.go [command]

# Install globally
go install
```

## Build and Test Commands

```bash
# Build
go build -o acon

# Build with version info
go build -ldflags "-X main.version=v1.0.0" -o acon

# Format code (required before commits)
gofmt -w .

# Run linter
golangci-lint run

# Run tests
go test ./...

# Run tests with race detector
go test -race ./...

# Run tests with coverage
go test -cover ./...

# Run specific package tests
go test -v ./internal/api
```

## Code Style Guidelines

### Formatting

- Always run `gofmt -w .` before committing
- Use tabs for indentation (Go standard)
- Maximum line length: 100-120 characters (soft limit)

### Naming Conventions

- Exported (public): `CamelCase` - e.g., `CreatePage`, `Client`
- Unexported (private): `camelCase` - e.g., `doRequest`, `appVersion`
- Package names: Short, lowercase, singular - e.g., `api`, `config`, `converter`
- Error variables: `err` for standard, `ErrSomething` for sentinels
- Acronyms: All caps in names - e.g., `APIToken`, `BaseURL`, `PageID`

### Go Idioms

- Ensure structs work with zero values where possible
- Check every error - never use `_` to ignore errors
- Wrap errors with context: `fmt.Errorf("context: %w", err)`
- Use pointers for mutation or large structs, values for small immutable data
- Favour struct embedding over inheritance
- Pass `context.Context` as first parameter for I/O or long-running operations
- Follow "accept interfaces, return structs" principle

### Package Structure

```
acon/
├── cmd/                    # Cobra commands (UI layer)
│   ├── root.go             # Root command and version handling
│   ├── page.go             # Page subcommands
│   ├── space.go            # Space subcommands
│   └── debug.go            # Debug command for troubleshooting
├── internal/
│   ├── api/                # Confluence REST API client
│   │   └── client.go
│   ├── config/             # Environment variable configuration
│   │   └── config.go
│   └── converter/          # Bidirectional Markdown conversion
│       ├── markdown.go     # Markdown → Confluence storage
│       └── storage.go      # Confluence storage → Markdown
├── .ai/                    # AI agent working files (DDD)
│   ├── projects/           # Project documents
│   ├── design/             # Design records
│   └── tasks/              # Task documentation
├── docs/                   # Human-facing documentation
├── testdata/               # Test fixtures
│   ├── comprehensive-test.md
│   ├── roundtrip-test.sh
│   └── README.md           # Feature support matrix
└── main.go                 # Entry point (version injection)
```

### Architecture Principles

- Separation of concerns: `cmd/` handles CLI, `internal/api/` handles API, `internal/converter/` handles conversion
- No circular dependencies: `cmd/` → `internal/*`, never the reverse
- Stateless API client: `Client` struct holds credentials, methods are pure operations

## Development Workflow

### Environment Setup

Required environment variables:

```bash
export CONFLUENCE_BASE_URL="https://your-instance.atlassian.net"
export CONFLUENCE_EMAIL="your-email@example.com"
export CONFLUENCE_API_TOKEN="your-api-token"  # or ATLASSIAN_API_TOKEN or JIRA_API_TOKEN
export CONFLUENCE_SPACE_KEY="YOUR_SPACE"      # optional default
```

Get an API token: <https://id.atlassian.com/manage-profile/security/api-tokens>

### Testing the CLI

```bash
./acon space list
./acon page list -s MYSPACE
echo "# Test" | ./acon page create -t "Test Page"
./acon page view PAGE_ID
./acon page view PAGE_ID -j
```

### Branch Management

- Main branch: `main`
- Feature branches: `feature/description` or `fix/description`
- Create PRs against `main`

### Commit Message Format

Follow conventional commits:

- `feat: add page deletion command`
- `fix: handle empty space key correctly`
- `docs: update README with examples`
- `refactor: simplify error handling`
- `test: add table-driven tests for converter`
- `chore: update dependencies`

## Testing Instructions

### Running Tests

```bash
go test ./...                  # Run all tests
go test -race ./...            # Check for race conditions
go test -cover ./...           # Check coverage
go test -v ./internal/api      # Verbose output for specific package
```

### Round-Trip Testing

```bash
# Automated round-trip test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grantcarthew/acon](https://github.com/grantcarthew/acon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
