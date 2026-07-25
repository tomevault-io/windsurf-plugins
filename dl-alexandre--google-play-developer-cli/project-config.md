---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**gpd** is a fast, lightweight CLI for the Google Play Developer Console - the Android equivalent to the App Store Connect CLI. It's designed to be AI-agent friendly with JSON-first output, predictable exit codes, and explicit flags.

Key characteristics:
- Sub-200ms cold start, minimal memory usage
- Comprehensive API coverage: publishing, reviews, analytics, monetization, vitals, purchases, permissions
- Cross-platform: macOS, Linux, Windows
- Secure credential storage with PII redaction
- Go 1.24.0

## Build & Test Commands

```bash
# Build
make build                    # Build for current platform
make build-all               # Build for all platforms (linux/amd64, linux/arm64, darwin/amd64, darwin/arm64, windows/amd64)

# Test
make test                    # Run all tests with race detection and coverage
make test-coverage           # Generate HTML coverage report
go test ./internal/auth      # Run tests for specific package
go test -v -run TestAuthStatus ./internal/auth  # Run single test

# Lint
make lint                    # Run golangci-lint (must be installed)

# Development
make run ARGS="version"      # Build and run with arguments
go run ./cmd/gpd auth status # Run directly without building

# Clean
make clean                   # Remove build artifacts and coverage reports
```

## Code Architecture

### Package Structure

The codebase follows a clean architecture with separation of concerns:

- **cmd/gpd/**: Entry point - minimal, just calls `cli.New().Execute()`
- **internal/api/**: Unified API client wrapper for Google Play APIs (Android Publisher, Play Developer Reporting, Games Management)
- **internal/auth/**: Authentication manager handling service accounts, OAuth, and credential sources
- **internal/cli/**: Cobra-based command definitions (32 command files)
- **internal/edits/**: Edit transaction lifecycle management with file locking and idempotency
- **internal/errors/**: Centralized error types with exit codes (0-8) and structured error responses
- **internal/output/**: Standardized JSON envelope structure with metadata
- **internal/storage/**: Platform-specific secure credential storage (Keychain/Secret Service/Credential Manager)
- **internal/config/**: Configuration file management
- **internal/logging/**: PII-redacting logger

### Authentication Flow

Authentication follows a priority chain (auth.Manager.Authenticate):

1. Explicit `--key` flag (keyfile)
2. `GPD_SERVICE_ACCOUNT_KEY` environment variable (JSON content)
3. `GOOGLE_APPLICATION_CREDENTIALS` environment variable (file path)
4. Application Default Credentials (ADC)

All authentication paths converge to `oauth2.TokenSource` with scopes for Android Publisher and Play Developer Reporting APIs.

### API Client Pattern

The `api.Client` uses lazy initialization with `sync.Once` for each API service:
- `AndroidPublisher()` - Publishing, reviews, monetization, purchases
- `PlayReporting()` - Analytics and vitals
- `GamesManagement()` - Games API

The client includes:
- Automatic retry with exponential backoff for 429/5xx errors
- Respect for Retry-After headers
- Semaphore-based concurrency control (default: 3 concurrent calls)
- Exclusive locking for upload operations

### Output Structure

All commands return a consistent JSON envelope (`output.Result`):

```json
{
  "data": { ... },
  "error": {
    "code": "AUTH_FAILURE",
    "message": "...",
    "hint": "...",
    "details": {}
  },
  "meta": {
    "noop": false,
    "durationMs": 150,
    "services": ["androidpublisher"],
    "nextPageToken": "...",
    "warnings": []
  }
}
```

Exit codes (errors/codes.go):
- 0: Success
- 1: General API error
- 2: Authentication failure
- 3: Permission denied
- 4: Validation error
- 5: Rate limited
- 6: Network error
- 7: Not found
- 8: Conflict

### Error Handling Pattern

Use structured errors from `internal/errors`:
- Create errors with `errors.NewAPIError(code, message)`
- Chain with `.WithHint()`, `.WithDetails()`, `.WithHTTPStatus()`
- Convert to output with `output.NewErrorResult(err)`
- Common errors are predefined: `ErrAuthNotConfigured`, `ErrPermissionDenied`, `ErrPackageRequired`, etc.

### Edit Transaction Management

The `edits.Manager` handles Google Play's edit transaction lifecycle:
- Creates/validates/commits edit sessions
- File-based locking with PID tracking (prevents concurrent edits)
- Idempotency store for deduplication
- Automatic cleanup of stale edits (7 day TTL, 1 hour idle TTL)
- Supports `--edit-id` for explicit edit IDs and `--no-auto-commit` for manual commits

### CLI Command Pattern

Commands in `internal/cli/` follow this structure:

1. Define cobra.Command with Use, Short, Long, RunE
2. Add flags specific to the command
3. RunE calls a method on the CLI struct (e.g., `c.authStatus(ctx)`)
4. Method implementation:
   - Parse/validate inputs
   - Call API via `c.apiClient`
   - Handle errors with structured error types
   - Return `output.Result` via `c.output.Write(result)`
5. CLI struct tracks startTime for duration metrics

### Testing Patterns

- Use table-driven tests for multiple scenarios

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dl-alexandre/Google-Play-Developer-CLI](https://github.com/dl-alexandre/Google-Play-Developer-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
