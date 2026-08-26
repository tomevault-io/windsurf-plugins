---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MAVT (Mobile App Version Tracker) is a Go application that monitors Apple App Store applications for version updates. It provides both CLI and daemon modes for tracking app versions over time.

**Key Features:**
- Track multiple iOS apps by bundle ID
- Detect and record version changes with release notes
- Web UI with App Store search for easy app discovery
- Run as a daemon for continuous monitoring
- Apprise integration for notifications (Discord, Slack, Telegram, email, 80+ services)
- REST API for programmatic access
- Docker containerization with GHCR images
- No database required - JSON file-based storage

## Development Commands

### Go Module Management
```bash
# Initialize Go module (if not already done)
go mod init github.com/username/mavt

# Download dependencies
go mod download

# Tidy dependencies
go mod tidy

# Verify dependencies
go mod verify
```

### Building
```bash
# Build the project
go build ./...

# Build specific package
go build ./path/to/package

# Build with output binary
go build -o mavt ./cmd/mavt
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run tests with race detector (important for concurrent code)
go test -race ./...

# Run tests with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...

# View coverage in browser
go tool cover -html=coverage.out

# Run specific test
go test -run TestName ./path/to/package

# Run tests matching a pattern
go test -run TestName.* ./...
```

**Note:** The codebase currently has no test files. When adding tests, place them alongside the code they test with `_test.go` suffix.

### Code Quality
```bash
# Format code
go fmt ./...

# Run go vet for static analysis
go vet ./...

# Run golangci-lint (if installed)
golangci-lint run
```

### Running
```bash
# Show version information
go run ./cmd/mavt -version

# Add an app to track
go run ./cmd/mavt -add com.apple.mobilesafari

# List all tracked apps
go run ./cmd/mavt -list

# Check for updates immediately
go run ./cmd/mavt -check

# Run as daemon (continuous monitoring)
go run ./cmd/mavt -daemon

# Show version history for an app
go run ./cmd/mavt -updates com.apple.mobilesafari

# Show recent updates (e.g., last 24 hours)
go run ./cmd/mavt -recent 24h

# Export data to a ZIP backup
go run ./cmd/mavt -export backup.zip

# Import data from a ZIP backup
go run ./cmd/mavt -import backup.zip

# Build and run the binary
go build -o mavt ./cmd/mavt
./mavt -version
./mavt -add com.apple.Music
```

### Docker Commands
```bash
# Build Docker image
docker build -t mavt:latest .

# Create named volume before first run (required by docker-compose)
docker volume create mavt-data

# Set proper permissions on volume (run as user 1000:1000)
docker run --rm -v mavt-data:/data alpine sh -c "chown -R 1000:1000 /data && ls -la /data"

# Run with docker-compose (daemon mode)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop container
docker-compose down

# Pull pre-built image from GHCR
docker pull ghcr.io/thomas/mavt:latest

# Run one-time check
docker run --rm -v $(pwd)/data:/app/data \
  -e MAVT_APPS=com.apple.mobilesafari \
  mavt:latest -check

# Interactive shell in container
docker-compose exec mavt sh
```

## Project Structure

```
mavt/
├── cmd/mavt/              # Main application entry point
│   └── main.go            # CLI commands, flag parsing, and daemon orchestration
├── internal/              # Private application code (Go convention: not importable)
│   ├── appstore/          # iTunes/App Store API client
│   │   └── client.go      # HTTP client for App Store lookups and search
│   ├── backup/            # Data backup and restore
│   │   └── backup.go      # ZIP export/import with path traversal and zip bomb protection
│   ├── config/            # Configuration management
│   │   └── config.go      # Environment variable loader (MAVT_* prefix)
│   ├── notifier/          # Apprise notification integration
│   │   └── notifier.go    # HTTP client for sending notifications
│   ├── server/            # HTTP server and web UI
│   │   └── server.go      # Web interface and REST API endpoints
│   ├── storage/           # Data persistence layer
│   │   └── storage.go     # JSON file-based storage with RWMutex
│   ├── tracker/           # Core tracking logic
│   │   └── tracker.go     # Version monitoring, comparison, and update detection
│   └── version/           # Version information
│       └── version.go     # Version constants (updated for releases)
├── pkg/models/            # Public data models
│   └── app.go             # AppInfo and VersionUpdate structs
├── data/                  # Runtime data directory (gitignored)
│   ├── apps/              # Current app information (one JSON per bundle ID)
│   └── updates/           # Version update history (array of updates per bundle ID)
├── Dockerfile             # Multi-stage Docker build
├── docker-compose.yml     # Docker Compose configuration (uses named volume)
└── .env.example           # Example environment variables
```

## Web Interface


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoiber/mavt](https://github.com/hoiber/mavt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
