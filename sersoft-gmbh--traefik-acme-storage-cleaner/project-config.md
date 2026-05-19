---
trigger: always_on
description: This is a small, focused Go CLI application that cleans expired certificates from Traefik ACME storage JSON files. The application is designed for simplicity and performance, processing multiple files in parallel using worker pools.
---

# Traefik ACME Storage Cleaner - Copilot Instructions

## Repository Overview

This is a small, focused Go CLI application that cleans expired certificates from Traefik ACME storage JSON files. The application is designed for simplicity and performance, processing multiple files in parallel using worker pools.

**Repository Statistics:**
- Language: Go 1.25
- Size: ~14 files (excluding dependencies)
- Type: CLI application
- Main Dependencies: github.com/traefik/traefik/v3 v3.6.7
- Test Coverage: 65.1%

## Project Structure

```
.
├── main.go                 # Main application code (single file application)
├── main_test.go            # Unit tests for main application
├── go.mod                  # Go module definition
├── go.sum                  # Go dependencies checksums
├── Dockerfile              # Multi-stage, multi-arch Docker build
├── README.md               # User documentation
├── LICENSE                 # License file
└── .github/
    ├── workflows/
    │   ├── build.yml       # CI build workflow (multi-OS, multi-arch)
    │   ├── test.yml        # CI test workflow (runs tests with coverage)
    │   ├── docker-publish.yml  # Docker image publishing on release
    │   └── enable-auto-merge.yml
    ├── dependabot.yml      # Dependabot configuration
    └── CODEOWNERS          # Code ownership configuration
```

## Building and Testing

### Prerequisites
- Go 1.25 or later (specified in go.mod)
- No additional tools required for basic builds

### Build Commands

**Standard build:**
```bash
go build .
```
This creates the `traefik-acme-storage-cleaner` binary in the current directory.

**Build time:** Approximately 2-3 minutes on first build (downloads dependencies), ~5-10 seconds on subsequent builds.

**Cross-platform builds:**
The project supports multiple platforms via environment variables:
```bash
# Linux AMD64 (default)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build .

# macOS ARM64
GOOS=darwin GOARCH=arm64 CGO_ENABLED=0 go build .

# Windows AMD64
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build .
```

**Important:** CGO_ENABLED=0 must be set to disable glibc linking for portability.

### Docker Build

**Build Docker image:**
```bash
docker build -t traefik-acme-storage-cleaner .
```

The Dockerfile uses a multi-stage, multi-architecture build:
1. Stage 1: golang:1.25-alpine builder with cross-compilation support
   - Uses `--platform=$BUILDPLATFORM` for native builder execution
   - Supports TARGETOS and TARGETARCH build arguments for cross-compilation
   - Supports linux/amd64 and linux/arm64 architectures
2. Stage 2: scratch base for minimal image size

**Build time:** ~3-5 minutes on first build, faster with Docker layer caching.

**Multi-arch build:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t traefik-acme-storage-cleaner .
```

### Testing

**Unit tests exist in main_test.go** with comprehensive test coverage.

**Run tests:**
```bash
go test -v ./...
```

**Run tests with coverage:**
```bash
go test -v -race -coverprofile=coverage.out -covermode=atomic ./...
go tool cover -func=coverage.out
```

**Test Coverage:** 65.1% (minimum threshold: 60%)

**Test coverage includes:**
- Core functions: `isExpired()`, `formatDomain()`, `defaultWorkers()`
- File processing: Expired cert removal, permission preservation, multi-resolver support, error handling
- Concurrency: Parallel file processing with configurable workers
- Edge cases: Empty files, invalid JSON, missing files, malformed certificates

**Test certificates:** Generated programmatically using `crypto/x509` with adjustable `NotBefore`/`NotAfter` timestamps. Helper function `mustGenerateTestCertificate()` ensures test failures on cert generation errors.

**Manual testing:**
1. Build the binary: `go build .`
2. Create a test ACME JSON file with sample certificates
3. Run: `./traefik-acme-storage-cleaner test-acme.json`
4. Verify the output shows certificate processing

### Running the Application

```bash
# Basic usage
./traefik-acme-storage-cleaner /path/to/acme.json

# Multiple files
./traefik-acme-storage-cleaner file1.json file2.json file3.json

# Custom worker count
./traefik-acme-storage-cleaner -workers 8 /path/to/acme.json

# Docker
docker run --rm -v /path/to/acme:/data ghcr.io/sersoft-gmbh/traefik-acme-storage-cleaner:latest /data/acme.json
```

## CI/CD Pipeline

### Build Workflow (.github/workflows/build.yml)
- Triggers on: push to main, pull requests to main
- Tests: Builds for 6 combinations (linux/darwin/windows × amd64/arm64)
- Go version: Read from go.mod using actions/setup-go@v6
- Each build runs with `CGO_ENABLED=0` and uses Go module caching

### Test Workflow (.github/workflows/test.yml)
- Triggers on: push to main, pull requests to main
- Runs tests with race detector: `go test -v -race -coverprofile=coverage.out -covermode=atomic ./...`
- Displays coverage breakdown: `go tool cover -func=coverage.out`
- Enforces 60% minimum coverage threshold
- Reports total coverage percentage
- Fails if coverage drops below threshold

### Docker Publish Workflow (.github/workflows/docker-publish.yml)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sersoft-gmbh/traefik-acme-storage-cleaner](https://github.com/sersoft-gmbh/traefik-acme-storage-cleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
