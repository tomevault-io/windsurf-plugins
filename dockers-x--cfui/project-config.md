---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CloudFlared UI (cfui) is a web-based control panel for managing Cloudflare Tunnel (cloudflared). It wraps the official cloudflared library and provides a web UI for configuration and control.

## Build & Development Commands

### Building the Binary
```bash
# Build with automatic version detection
make build

# Build with specific version
VERSION=v1.0.0 make build

# Manual build (if Make unavailable)
CGO_ENABLED=0 go build -trimpath \
  -ldflags="-s -w -X 'cfui/version.Version=dev' -X 'cfui/version.BuildTime=$(date -u '+%Y-%m-%d_%H:%M:%S_UTC')' -X 'cfui/version.GitCommit=$(git rev-parse --short HEAD)'" \
  -o cfui .
```

### Running Tests
```bash
make test
# Or directly
go test -v ./...
```

### Running Locally
```bash
# Build and run
make run

# Or run directly without building
go run main.go

# Set custom port
PORT=8080 go run main.go

# Set custom data directory
DATA_DIR=/custom/path go run main.go
```

### Docker Build
```bash
# Build Docker image
make build-docker

# Or manually with version info
VERSION=$(git describe --tags --always --dirty 2>/dev/null || echo "dev")
BUILD_TIME=$(date -u '+%Y-%m-%d_%H:%M:%S_UTC')
GIT_COMMIT=$(git rev-parse --short HEAD 2>/dev/null || echo "unknown")

docker build \
  --build-arg VERSION=${VERSION} \
  --build-arg BUILD_TIME=${BUILD_TIME} \
  --build-arg GIT_COMMIT=${GIT_COMMIT} \
  -t cfui:${VERSION} -t cfui:latest .
```

### Version Information
```bash
# View current version info
make version
```

## Architecture

### Core Components

**main.go**: Application entry point that orchestrates initialization.
- Sets up config directory (env: `DATA_DIR`, default: `~/.cloudflared-web`)
- Initializes config manager, runner, and server
- Embeds web UI assets and locale files

**config/** (config/config.go): Configuration management with thread-safe operations.
- Manages `data/config.json` persistence
- Handles all cloudflared parameters (protocol, region, metrics, etc.)
- Provides default values and atomic read/write operations via mutex

**service/** (service/runner.go): Cloudflared tunnel lifecycle management.
- Wraps official `github.com/cloudflare/cloudflared` library
- Handles start/stop/status operations with graceful shutdown
- Implements auto-restart logic with exponential backoff (5s → 60s max)
- Creates temporary YAML config files for custom tags
- Intercepts CLI exit behavior to prevent process termination
- Distinguishes between retryable (network) and non-retryable (auth) errors

**server/** (server/server.go, server/middleware.go): HTTP server and API handlers.
- Serves embedded static web UI from `web/dist/`
- Provides REST API endpoints for config, status, and control
- Serves i18n translations from embedded TOML files
- Middleware for panic recovery and request logging

**logger/** (logger/logger.go): Structured logging with rotation.
- Uses `go.uber.org/zap` for structured logging
- Implements file rotation via `lumberjack`
- Logs to both file (`~/.cloudflared-web/logs/cfui.log`) and console
- JSON format for files, colored console output

**version/**: Version information injected at build time via ldflags.

### Critical Architecture Details

**Cloudflared Integration**: The app uses the official cloudflared library as a dependency (not spawning external process). This means:
- `tunnel.Init()` must be called exactly once via `sync.Once`
- CLI framework is intercepted to prevent `os.Exit()` calls
- Metrics registration errors require full process restart (not just tunnel restart)
- Custom tags require temporary YAML config files (cleaned up on shutdown)

**Auto-Restart Logic**:
- Enabled by default via `Config.AutoRestart`
- Exponential backoff: 5s, 10s, 20s, 40s (max 60s)
- Max 10 restart attempts
- Restart counter resets after 5 minutes of uptime
- Non-retryable errors (auth, config) skip auto-restart

**Graceful Shutdown**:
- 30-second timeout for tunnel shutdown
- Context cancellation propagates to cloudflared
- Temporary config files cleaned up via defer
- Stop endpoint responds immediately, then stops async to prevent connection errors

**Thread Safety**:
- Config manager uses `sync.RWMutex` for concurrent access
- Runner uses mutex for state management (running, lastError, etc.)

## Environment Variables

- `BIND_HOST`: Web server bind address (default: `0.0.0.0`)
- `PORT`: Web server port (default: `14333`)
- `DATA_DIR`: Data directory for config (default: `./data`, Docker: `/app/data`)
- `LOG_DIR`: Log directory (default: `{DATA_DIR}/logs`, Docker: `/app/logs`)
- `LOG_LEVEL`: Log level - debug, info, warn, error (default: `info`)

## API Endpoints

- `GET /api/config` - Get current configuration
- `POST /api/config` - Update configuration
- `GET /api/status` - Get tunnel running status and last error
- `POST /api/control` - Control tunnel (action: "start" | "stop")
- `GET /api/i18n/{lang}` - Get translations (en, zh, ja)

## Configuration File Structure

Located at `{DATA_DIR}/config.json`:

```json
{
  "token": "cloudflare-tunnel-token",
  "auto_start": false,
  "auto_restart": true,
  "custom_tag": "custom-identifier",
  "protocol": "auto",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dockers-x/cfui](https://github.com/dockers-x/cfui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
