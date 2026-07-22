---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Armis CLI is an enterprise-grade security scanning tool written in Go that integrates with Armis Cloud. It scans repositories and container images for security vulnerabilities, secrets, and license risks.

**Prerequisites:** Go 1.25+, [golangci-lint](https://golangci-lint.run/) v2.0+, Make

## Build Commands

```bash
make build          # Build binary to bin/armis-cli
make install        # Install binary to /usr/local/bin (or PREFIX)
make test           # Run all tests with verbose output (uses gotestsum if installed)
make lint           # Run golangci-lint
make clean          # Remove build artifacts
make release        # Build for all platforms (linux/darwin/windows, amd64/arm64)
make scan           # Run security scan on this repository (requires built binary)
make tools          # Install dev tools (gotestsum)
```

Run a single test:

```bash
go test -v ./internal/api -run TestClientStartIngest
go test -v ./internal/output/... -run TestHumanFormatter
```

## Architecture

### Entry Point and Command Structure

- `cmd/armis-cli/main.go` - Entry point. Sets version info, initializes colors via `cli.InitColors()`, calls `cmd.Execute()`.
- `internal/cmd/` - Cobra command definitions:
  - `root.go` - Root command with global flags. `PersistentPreRunE` initializes color mode, syncs output styles, and starts background update check. `getAuthProvider()` delegates to `auth.NewAuthProvider()`.
  - `scan.go` - Parent scan command with shared flags. Its `PersistentPreRunE` manually chains to `rootCmd.PersistentPreRunE` (Cobra does not auto-chain when a child also defines `PersistentPreRunE`).
  - `scan_repo.go` - Repository scanning subcommand
  - `scan_image.go` - Container image scanning subcommand (`--tarball` flag for pre-exported images)
  - `auth.go` - Standalone `auth` command for testing JWT authentication (prints raw token)
  - `context.go` - Signal handling: `NewSignalContext()` creates context canceled on SIGINT/SIGTERM

### Core Packages

- `internal/auth/` - Authentication provider supporting two modes. JWT (priority): client credentials exchange at `/api/v1/auth/token`, auto-refresh 5min before expiry, tenant ID extracted from `customer_id` JWT claim. Basic (fallback): static token + explicit tenant ID. Implements `AuthHeaderProvider` interface used by the API client.
- `internal/api/` - API client for Armis Cloud. Two HTTP clients: one for general calls (60s timeout), one for uploads (streaming, no timeout, no retry). Functional options pattern (`WithHTTPClient()`, `WithUploadHTTPClient()`, `WithAllowLocalURLs()`). Upload uses `io.Pipe` streaming to avoid OOM on large files. Enforces HTTPS, validates presigned S3 URLs against SSRF.
- `internal/model/` - Data structures: `Finding` (23 fields), `ScanResult`, `Summary`, `Fix`, `FindingValidation` (with taint/reachability analysis), API response types (`NormalizedFinding`, pagination).
- `internal/output/` - Output formatters (human, json, sarif, junit) implementing the `Formatter` interface. `styles.go` defines ~50 lipgloss styles using Tailwind CSS color palette. `icons.go` defines Unicode constants (severity dots, box-drawing chars). `SyncColors()` switches between full-color and plain styles based on `cli.ColorsEnabled()`.
- `internal/cli/` - Centralized color state management. `InitColors(mode)` resolves `--color` flag (auto/always/never) with `NO_COLOR` env, `TERM=dumb`, and TTY detection on stderr. `PrintError()`/`PrintWarning()` parse JSON `{"detail":"..."}` from API errors for clean display.
- `internal/scan/repo/` - Repository scanner: creates tar.gz (with `.armisignore` support via go-git gitignore matcher), uploads, polls, fetches paginated results. Builder pattern with `WithPollInterval()`, `WithIncludeFiles()`, `WithSBOMVEXOptions()`.
- `internal/scan/image/` - Image scanner: validates image names via `distribution/reference`, uses docker/podman to export, then uploads. Also supports direct tarball scanning.
- `internal/scan/` - Shared scan utilities: `status.go` (status formatting, severity mapping), `finding_type.go` (classifies findings as VULNERABILITY/SCA/SECRET/MISCONFIG/LICENSE), `sbom_vex.go` (downloads SBOM/VEX from presigned S3 URLs).
- `internal/progress/` - Braille-dot spinner with timer display, lipgloss styling, cursor hiding, CI detection (auto-disables animation). Context-aware with configurable timeout (default 30min). Upload progress via `NewReader()`/`NewWriter()` wrappers.
- `internal/update/` - Background version checker against GitHub Releases API with file-based caching (~/.cache/armis-cli/, 24h TTL). Semver comparison. Skipped in CI, dev builds.
- `internal/httpclient/` - HTTP client with exponential backoff retry (cenkalti/backoff). Retries on 5xx errors.
- `internal/util/` - Path sanitization (`SanitizePath`, `SafeJoinPath` for traversal prevention), secret masking (12 regex patterns), category formatting.

### Key Interfaces

- `output.Formatter` - `Format()` and `FormatWithOptions()` for all output formatters
- `api.AuthHeaderProvider` - `GetAuthorizationHeader(ctx)` decouples auth from API client

### Scan Flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArmisSecurity/armis-cli](https://github.com/ArmisSecurity/armis-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
