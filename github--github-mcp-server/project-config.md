---
trigger: always_on
description: This is the **GitHub MCP Server**, a Model Context Protocol (MCP) server that connects AI tools to GitHub's platform. It enables AI agents to manage repositories, issues, pull requests, workflows, and more through natural language.
---

# GitHub MCP Server - Copilot Instructions

## Project Overview

This is the **GitHub MCP Server**, a Model Context Protocol (MCP) server that connects AI tools to GitHub's platform. It enables AI agents to manage repositories, issues, pull requests, workflows, and more through natural language.

**Key Details:**
- **Language:** Go 1.24+ (~38k lines of code)
- **Type:** MCP server application with CLI interface
- **Primary Package:** github-mcp-server (stdio MCP server - **this is the main focus**)
- **Secondary Package:** mcpcurl (testing utility - don't break it, but not the priority)
- **Framework:** Uses modelcontextprotocol/go-sdk for MCP protocol, google/go-github for GitHub API
- **Size:** ~60MB repository, 70 Go files
- **Library Usage:** This repository is also used as a library by the remote server. Functions that could be called by other repositories should be exported (capitalized), even if not required internally. Preserve existing export patterns.

**Code Quality Standards:**
- **Popular Open Source Repository** - High bar for code quality and clarity
- **Comprehension First** - Code must be clear to a wide audience
- **Clean Commits** - Atomic, focused changes with clear messages
- **Structure** - Always maintain or improve, never degrade
- **Code over Comments** - Prefer self-documenting code; comment only when necessary

## Critical Build & Validation Steps

### Required Commands (Run Before Committing)

**ALWAYS run these commands in this exact order before using report_progress or finishing work:**

1. **Format Code:** `script/lint` (runs `gofmt -s -w .` then `golangci-lint`)
2. **Run Tests:** `script/test` (runs `go test -race ./...`)
3. **Update Documentation:** `script/generate-docs` (if you modified MCP tools/toolsets)

**These commands are FAST:** Lint ~1s, Tests ~1s (cached), Build ~1s

### When Modifying MCP Tools/Endpoints

If you change any MCP tool definitions or schemas:
1. Run tests with `UPDATE_TOOLSNAPS=true go test ./...` to update toolsnaps
2. Commit the updated `.snap` files in `pkg/github/__toolsnaps__/`
3. Run `script/generate-docs` to update README.md
4. Toolsnaps document API surface and ensure changes are intentional

### Common Build Commands

```bash
# Download dependencies (rarely needed - usually cached)
go mod download

# Build the server binary
go build -v ./cmd/github-mcp-server

# Run the server
./github-mcp-server stdio

# Run specific package tests
go test ./pkg/github -v

# Run specific test
go test ./pkg/github -run TestGetMe
```

## Project Structure

### Directory Layout

```
.
├── cmd/
│   ├── github-mcp-server/    # Main MCP server entry point (PRIMARY FOCUS)
│   └── mcpcurl/              # MCP testing utility (secondary - don't break it)
├── pkg/                      # Public API packages
│   ├── github/               # GitHub API MCP tools implementation
│   │   └── __toolsnaps__/    # Tool schema snapshots (*.snap files)
│   ├── toolsets/             # Toolset configuration & management
│   ├── errors/               # Error handling utilities
│   ├── sanitize/             # HTML/content sanitization
│   ├── log/                  # Logging utilities
│   ├── raw/                  # Raw data handling
│   ├── buffer/               # Buffer utilities
│   └── translations/         # i18n translation support
├── internal/                 # Internal implementation packages
│   ├── ghmcp/                # GitHub MCP server core logic
│   ├── githubv4mock/         # GraphQL API mocking for tests
│   ├── toolsnaps/            # Toolsnap validation system
│   └── profiler/             # Performance profiling
├── e2e/                      # End-to-end tests (require GitHub PAT)
├── script/                   # Build and maintenance scripts
├── docs/                     # Documentation
├── .github/workflows/        # CI/CD workflows
└── [config files]            # See below
```

### Key Configuration Files

- **go.mod / go.sum:** Go module dependencies (Go 1.24.0+)
- **.golangci.yml:** Linter configuration (v2 format, ~15 linters enabled)
- **Dockerfile:** Multi-stage build (golang:1.25.8-alpine → distroless)
- **server.json:** MCP server metadata for registry
- **.goreleaser.yaml:** Release automation config
- **.gitignore:** Excludes bin/, dist/, vendor/, *.DS_Store, github-mcp-server binary

### Important Scripts (script/ directory)

- **script/lint** - Runs `gofmt` + `golangci-lint`. **MUST RUN** before committing
- **script/test** - Runs `go test -race ./...` (full test suite)
- **script/generate-docs** - Updates README.md tool documentation. Run after tool changes
- **script/licenses** - Updates third-party license files when dependencies change
- **script/licenses-check** - Validates license compliance (runs in CI)
- **script/get-me** - Quick test script for get_me tool
- **script/get-discussions** - Quick test for discussions
- **script/tag-release** - **NEVER USE THIS** - releases are managed separately

## GitHub Workflows (CI/CD)

All workflows run on push/PR unless noted. Located in `.github/workflows/`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [github/github-mcp-server](https://github.com/github/github-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
