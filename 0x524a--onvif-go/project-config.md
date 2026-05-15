---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

onvif-go is a production-ready Go library for communicating with ONVIF-compliant IP cameras. It provides both a client library for camera control and a server implementation for camera simulation/testing.

**Key Features:**
- ONVIF client with 200+ APIs across Device, Media, PTZ, and Imaging services
- ONVIF server for virtual camera simulation
- WS-Discovery for network camera detection
- WS-Security authentication with digest passwords
- Multiple CLI tools for camera interaction and diagnostics

## Essential Commands

### Build
```bash
# Build all CLI tools for current platform
make build

# Build for multiple platforms (Linux, Windows, macOS)
make build-all

# Build specific CLI tool
go build -o bin/onvif-cli ./cmd/onvif-cli
```

### Test
```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -v -race -coverprofile=coverage.out ./...
make test-coverage

# Run benchmarks
make bench
go test -bench=. -benchmem ./...

# Run specific package tests
go test -v ./discovery
go test -v ./server
```

### Lint and Format
```bash
# Run all checks (fmt, vet, lint)
make check

# Format code
make fmt

# Run linter
make lint  # Requires golangci-lint
```

### Development
```bash
# Install dependencies
make deps

# Clean build artifacts
make clean

# Build examples
make examples

# Run CLI tools
./bin/onvif-cli
./bin/onvif-quick
```

### CLI Tools

**onvif-cli**: Comprehensive ONVIF client with interactive and non-interactive modes
```bash
# Interactive menu
./bin/onvif-cli

# Discover cameras
./bin/onvif-cli discover -interface eth0 -timeout 5

# Get device info
./bin/onvif-cli -op info -endpoint http://camera-ip/onvif/device_service -username admin -password pass
```

**onvif-diagnostics**: Camera testing and XML capture for debugging
```bash
./bin/onvif-diagnostics -endpoint http://camera-ip/onvif/device_service -username admin -password pass -verbose

# Capture raw SOAP XML
./bin/onvif-diagnostics ... -capture-xml
```

**onvif-server**: Virtual camera server for testing
```bash
./bin/onvif-server -profiles 5 -username admin -password mypass -port 9000
```

## Architecture

### Package Structure

```
onvif-go/
├── *.go                    # Core client library (client.go, device.go, media.go, ptz.go, imaging.go, etc.)
├── types.go                # ONVIF type definitions (all SOAP XML structures)
├── internal/soap/          # SOAP client with WS-Security (NOT exported)
├── discovery/              # WS-Discovery implementation (exported package)
├── server/                 # ONVIF server implementation (exported package)
├── cmd/                    # CLI tools
│   ├── onvif-cli/         # Full-featured client
│   ├── onvif-quick/       # Lightweight tool
│   ├── onvif-diagnostics/ # Debugging and XML capture
│   ├── onvif-server/      # Server CLI
│   └── generate-tests/    # Test generation from XML captures
├── testing/               # Test utilities (mock_server.go)
├── testdata/captures/     # Real camera SOAP response captures
└── examples/              # Usage examples
```

### Key Components

**Client Layer** (`client.go`):
- Main `Client` struct with HTTP connection pooling
- Functional options pattern for configuration (WithCredentials, WithTimeout, WithHTTPClient)
- Context-aware operations throughout
- Thread-safe credential management with sync.RWMutex

**Service Implementations**:
- `device.go` + `device_*.go`: 98 Device Management APIs (configuration, users, network, certificates, WiFi, storage)
- `media.go`: Media profiles, stream URIs (RTSP/HTTP), snapshots, encoder configuration
- `ptz.go`: PTZ control (continuous, absolute, relative movement, presets)
- `imaging.go`: Image settings (brightness, contrast, exposure, focus, white balance)
- `event.go`: Event service (subscriptions, pull-point)
- `deviceio.go`: Device I/O and relay control

**SOAP Layer** (`internal/soap/`):
- WS-Security UsernameToken authentication with password digest (SHA-1)
- XML marshaling/unmarshaling for ONVIF SOAP messages
- Error handling with ONVIFError type
- NOT exported - internal implementation detail

**Discovery** (`discovery/`):
- WS-Discovery multicast probe on 239.255.255.250:3702
- Network interface selection support
- Device deduplication by endpoint reference

**Server** (`server/`):
- Virtual multi-lens camera simulator
- Implements Device, Media, PTZ, and Imaging services
- Configurable number of camera profiles (up to 10)
- WS-Security authentication support

### Type System

All ONVIF types are defined in `types.go` (~30,000+ lines). Key patterns:

- XML struct tags for SOAP serialization
- Pointer fields for optional values (ONVIF convention)
- Namespace-aware XML marshaling
- Comprehensive coverage of ONVIF Core, Device, Media, PTZ, Imaging specs

## Development Patterns

### Client Usage Pattern
```go
// 1. Create client with options
client, err := onvif.NewClient(
    endpoint,
    onvif.WithCredentials(username, password),
    onvif.WithTimeout(30*time.Second),
)

// 2. Initialize to discover service endpoints
if err := client.Initialize(ctx); err != nil {
    return err
}

// 3. Use service methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0x524a/onvif-go](https://github.com/0x524a/onvif-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
