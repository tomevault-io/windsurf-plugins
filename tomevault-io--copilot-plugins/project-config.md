---
trigger: always_on
description: > This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

## claude-gate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Gate is a high-performance Go OAuth proxy for Anthropic's Claude API that enables FREE Claude usage for Pro/Max subscribers by identifying as "Claude Code" (Anthropic's official CLI). This is the official Go port of the original Python implementation.

**Key Features:**
- OAuth 2.0 PKCE authentication flow
- Secure token storage with OS keychain integration (planned)
- Interactive TUI dashboard for monitoring
- Cross-platform support (macOS, Linux, Windows)
- Multiple distribution methods (NPM, Homebrew, direct binary)

## Quick Navigation

### Core Implementation Files
- Main entry: `cmd/claude-gate/main.go`
- OAuth client: `internal/auth/client.go`
- Proxy handler: `internal/proxy/handler.go`
- Dashboard: `internal/ui/dashboard/dashboard.go`

### Configuration Files
- Go module: `go.mod`
- Build config: `.goreleaser.yml`
- CI/CD: `.github/workflows/release.yml`
- NPM package: `npm/package.json`

### Test Files
- Unit tests: `*_test.go` (alongside source)
- Integration: `internal/test/integration/*_test.go`
- E2E: `internal/test/e2e/*_test.go`
- Test helpers: `internal/test/helpers/helpers.go`

### Scripts
- Version update: `scripts/update-version.sh`
- NPM test: `scripts/test-npm-local.sh`
- Add co-author: `.claude/scripts/add-claude-coauthor.sh`

## Common Commands

### Building
```bash
make build        # Build for current platform
make snapshot     # Build all platforms (uses GoReleaser)
make install      # Install to ~/bin
```

### Testing
```bash
make test              # Run unit tests with coverage
make test-unit         # Run unit tests only (short mode)
make test-integration  # Run integration tests
make test-e2e          # Run end-to-end tests
make test-all          # Run all test types
make npm-test          # Test NPM package locally
go test -v ./...       # Quick test during development
```

### Running
```bash
claude-gate start --host 127.0.0.1 --port 5789  # Start proxy server
claude-gate dashboard                            # Start with interactive dashboard
claude-gate auth login                           # Authenticate with Claude
```

### Releasing
```bash
make release VERSION=0.2.0  # Create new release
./scripts/update-version.sh # Update version in all files
```

## Architecture

The codebase follows clean architecture principles with clear separation of concerns:

### Core Components

1. **CLI Layer** (`cmd/claude-gate/`)
   - Uses Kong framework for command parsing
   - Entry point for all operations

2. **Auth Package** (`internal/auth/`)
   - OAuth 2.0 PKCE implementation
   - Token storage and management
   - Browser automation for login flow

3. **Proxy Package** (`internal/proxy/`)
   - HTTP proxy server implementation
   - Request/response transformation
   - Enhanced server with monitoring capabilities

4. **UI Package** (`internal/ui/`)
   - Bubble Tea-based TUI components
   - Interactive dashboard for monitoring
   - Reusable components (spinner, progress, styles)

### Request Flow
1. Client connects to local proxy (default: 127.0.0.1:5789)
2. Proxy validates authentication token
3. Request transformed to identify as "Claude Code"
4. Forwarded to Claude API with OAuth credentials
5. Response streamed back to client

### Security Model
- OAuth 2.0 PKCE flow for authentication
- Tokens stored securely (keychain integration planned)
- Local-only proxy binding by default
- Optional proxy authentication token for additional security

## Project Structure

```
claude-gate/
├── cmd/
│   └── claude-gate/          # CLI application entry point
│       ├── main.go          # Main entry point with Kong CLI setup
│       └── auth_storage.go  # Storage-related CLI commands
├── internal/                 # Private application code (Go convention)
│   ├── auth/                # Authentication & token management
│   │   ├── client.go        # OAuth client implementation
│   │   ├── oauth.go         # OAuth flow logic
│   │   ├── storage_*.go     # Various storage backends
│   │   └── *_test.go        # Unit tests
│   ├── config/              # Configuration management
│   ├── proxy/               # Proxy server implementation
│   │   ├── handler.go       # Main proxy handler
│   │   ├── server.go        # Enhanced server with monitoring
│   │   └── transformer.go   # Request/response transformation
│   ├── test/                # Test infrastructure
│   │   ├── integration/     # Integration tests (build tag: integration)
│   │   ├── e2e/            # End-to-end tests (build tag: e2e)
│   │   ├── helpers/        # Shared test utilities
│   │   └── testdata/       # Test fixtures
│   └── ui/                  # Terminal UI components
│       ├── components/      # Reusable UI components
│       ├── dashboard/       # Interactive dashboard
│       ├── styles/         # Terminal styling (Lipgloss)
│       └── utils/          # UI utilities
├── docs/                    # Project documentation
│   ├── architecture/       # Architecture decisions and diagrams

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/copilot-plugins](https://github.com/tomevault-io/copilot-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
