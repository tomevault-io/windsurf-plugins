---
trigger: always_on
description: Tarmac is a Go-based framework for building serverless functions using WebAssembly (WASM). It supports multiple programming languages including Go, Rust, Zig, and others, enabling developers to write functions in their preferred language while leveraging Tarmac's robust distributed service capabilities.
---

# Tarmac Framework Development Instructions

Tarmac is a Go-based framework for building serverless functions using WebAssembly (WASM). It supports multiple programming languages including Go, Rust, Zig, and others, enabling developers to write functions in their preferred language while leveraging Tarmac's robust distributed service capabilities.

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Environment Setup
- Install Go 1.23+ (tested with Go 1.24.6): `go version` to verify
- Install TinyGo for WASM compilation (REQUIRED):
  - Download latest version: `wget https://github.com/tinygo-org/tinygo/releases/download/v0.39.0/tinygo_0.39.0_amd64.deb`
  - Install: `sudo dpkg -i tinygo_0.39.0_amd64.deb`
  - Verify: `tinygo version` (should show 0.39.0 or later)
- Install golangci-lint for code quality: `curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin latest`
- Ensure Docker and Docker Compose are available for integration testing

### Build Process
- **Build all test WASM modules**: `make build`
  - **Duration: ~52 seconds** - NEVER CANCEL. Set timeout to 90+ seconds.
  - Builds 10 test WebAssembly modules using TinyGo
  - Required before running any tests
- **Build main Tarmac application**: `go build ./cmd/tarmac`
  - **Duration: ~41 seconds** - NEVER CANCEL. Set timeout to 60+ seconds.
  - Downloads dependencies and compiles the main server binary
- **Build example WASM functions**: 
  - Navigate to example directory: `cd example/tac/go` or `cd example/echo/go`
  - Build: `make build`
  - **Duration: ~6 seconds per example** - Set timeout to 30+ seconds.

### Testing
- **Unit tests**: Individual package tests work but may fail if they require external services
  - Example: `go test -v ./pkg/callbacks/` (quick, no external deps)
  - **Avoid**: `go test -v ./pkg/app/` (requires Redis/databases)
- **Integration tests via Docker Compose**:
  - Base tests: `make tests-base`
  - **Duration: 3-4 minutes** - NEVER CANCEL. Set timeout to 10+ minutes.
  - Database-specific tests: `make tests-redis`, `make tests-mysql`, `make tests-postgres`, `make tests-cassandra`, `make tests-boltdb`, `make tests-inmemory`
  - **Duration: 3-5 minutes each** - NEVER CANCEL. Set timeout to 10+ minutes per test suite.

### Code Quality and Linting
- **Go vet**: `go vet -v ./...`
  - **Duration: ~6 seconds** - Set timeout to 30+ seconds.
  - Should pass cleanly
- **golangci-lint**: `golangci-lint run -E misspell -E revive -E bodyclose -E errname --timeout=5m`
  - **Duration: ~14 seconds** - Set timeout to 10+ minutes (as specified in command).
  - May show existing lint issues in codebase (expected)
- **Go formatting**: `gofmt -d .` to check formatting, `gofmt -w .` to fix

### Running Tarmac
- **Development mode** (single function):
  ```bash
  # Build a WASM function first
  cd example/tac/go && make build
  
  # Run Tarmac with the function
  cd /path/to/repo/root
  APP_ENABLE_TLS=false APP_LISTEN_ADDR=0.0.0.0:8080 APP_WASM_FUNCTION=example/tac/go/functions/tarmac.wasm APP_ENABLE_KVSTORE=false ./tarmac
  ```
- **With Docker Compose** (full stack with databases):
  ```bash
  # In example directory
  cd example/tac/go
  make build
  docker compose up
  ```
- **Production mode**: Use multi-function configuration with tarmac.json config file

## Validation Scenarios

### Always Test After Making Changes
1. **Build validation**:
   ```bash
   make build  # Should complete in ~52 seconds
   go build ./cmd/tarmac  # Should complete in ~41 seconds
   ```

2. **Basic functionality test**:
   ```bash
   # Build an example
   cd example/tac/go && make build
   
   # Start Tarmac (in background or separate terminal)
   cd ../../../ && APP_ENABLE_TLS=false APP_LISTEN_ADDR=0.0.0.0:8080 APP_WASM_FUNCTION=example/tac/go/functions/tarmac.wasm APP_ENABLE_KVSTORE=false ./tarmac
   
   # Test HTTP endpoint (should get response, even if 500 due to config)
   curl -X POST --data "test" http://localhost:8080/ -v
   ```

3. **Code quality validation**:
   ```bash
   go vet -v ./...  # Should pass
   golangci-lint run -E misspell -E revive -E bodyclose -E errname --timeout=5m  # May show existing issues
   ```

### Manual Testing Scenarios
- **Example function testing**: Build and run example functions to verify WebAssembly integration works
- **HTTP endpoint testing**: Verify Tarmac starts HTTP server and responds to requests
- **Configuration testing**: Test with different environment variables (TLS on/off, different ports, KV store configs)

## Project Structure Navigation

### Key Directories
- `cmd/tarmac/`: Main application entry point
- `pkg/app/`: Core Tarmac server application logic
- `pkg/callbacks/`: Host callback implementations (KV store, SQL, HTTP client, etc.)
- `pkg/wasm/`: WebAssembly module management and execution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarmac-project/tarmac](https://github.com/tarmac-project/tarmac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
