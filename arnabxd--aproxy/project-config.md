---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AProxy is an anonymous proxy server written in Go that scrapes free proxies from various sources, validates their health, and provides a rotating proxy service with privacy features.

## Development Commands

```bash
# Build the application
go build -o aproxy ./cmd/aproxy

# Run the application
./aproxy

# Run with custom config
./aproxy -config config.json

# Generate default config
./aproxy -gen-config

# Run with .env file (copy .env.example to .env first)
cp .env.example .env
./aproxy

# Run tests
go test ./...

# Format code
go fmt ./...

# Run linter (if available)
golangci-lint run

# Tidy dependencies
go mod tidy
```

## Docker Development

```bash
# Build and run with Docker Compose
docker-compose up --build

# Run in background
docker-compose up -d

# View logs
docker-compose logs -f

# Stop containers
docker-compose down

# Build Docker image manually
docker build -t aproxy .

# Run with custom environment
cp .env.docker .env
docker-compose up
```

## Project Architecture

### Core Components

- **Scraper** (`pkg/scraper/`): Fetches proxy lists from multiple sources (ProxyScrape, FreeProxyList, ProxyListOrg, GitHub)
- **Checker** (`pkg/checker/`): Validates proxy health with SQLite caching, intelligent check intervals, and unified logging
- **Manager** (`pkg/manager/`): Manages proxy pool with database persistence, in-memory cache, and auto-refresh
- **Database** (`internal/database/`): SQLite-based persistent storage with sqlc-generated type-safe queries
- **Proxy Server** (`pkg/proxy/`): HTTP/HTTPS proxy server with privacy features
- **Config** (`internal/config/`): Advanced configuration management with Viper and validation support

### Key Features

- **Non-blocking startup**: Server starts immediately with cached proxies, background checking builds proxy pool progressively
- **Multi-source scraping**: Aggregates proxies from multiple free proxy services
- **SQLite database**: Persistent proxy storage with intelligent caching (10-minute check intervals)
- **Progressive health checking**: Checks proxies in small batches with delays to avoid overwhelming system
- **Smart health monitoring**: Reduces redundant API calls with configurable check intervals
- **Rotating proxy**: Round-robin and random proxy selection with database persistence
- **Privacy protection**: Header stripping, user-agent spoofing, connection sanitization
- **Authentication**: Bearer token authentication for secure proxy access
- **HTTPS support**: CONNECT method tunneling for secure connections
- **Database statistics**: Real-time proxy pool, database, and server metrics via `/stats` endpoint
- **Background operations**: All proxy scraping and checking happens in background without blocking server
- **Performance optimized**: Hybrid in-memory + database storage for fast access
- **Docker support**: Production-ready containerization with persistent volumes
- **Configuration validation**: Comprehensive validation with helpful error messages

### Authentication & Security

**Authentication (✅ IMPLEMENTED):**
- Bearer token authentication using `server.auth_token` config
- Client authentication via `Proxy-Authorization: Bearer <token>` header
- Environment variable support: `APROXY_SERVER_AUTH_TOKEN`
- Returns 407 Proxy Authentication Required for invalid/missing tokens
- Authentication failures logged with client IP addresses

**Privacy Features:**
- Strips identifying headers (X-Forwarded-For, X-Real-IP, etc.)
- Adds spoofed User-Agent headers
- Removes server identification headers from responses
- Supports HTTPS tunneling for encrypted connections

**Security Considerations:**
- ⚠️ TLS verification disabled (`InsecureSkipVerify: true`) for upstream connections
- ⚠️ SQLite database stored unencrypted at rest
- ⚠️ Authentication tokens stored in plain text
- ⚠️ No built-in rate limiting for authentication attempts
- ⚠️ Limited audit logging for security events

### Configuration

AProxy uses **Viper** for advanced configuration management with validation:

**Configuration Sources (in priority order):**
1. **Command line flags**: `-config`, `-gen-config`, `-version`
2. **Environment variables**: All settings can be overridden with `APROXY_` prefix
3. **Config files**: YAML, JSON, TOML supported (searches `./`, `./config/`, `/etc/aproxy/`)
4. **Defaults**: Sensible defaults for all settings

**Supported Scraper Sources:** (all are plain `host:port` / `proto://host:port` text lists)
- `proxyscrape`: ProxyScrape API
- `freeproxylist`: FreeProxyList scraper
- `proxylistorg`: ProxyListOrg scraper
- `github`: GitHub proxy list scraper (proxifly/free-proxy-list)

**Configuration Management:**
```bash
# Generate sample config file
./aproxy -gen-config  # Creates config.yaml

# Run with specific config
./aproxy -config myconfig.yaml

# Use environment variables (Docker-friendly)
export APROXY_SERVER_LISTEN_ADDR=":9090"
export APROXY_DATABASE_PATH="/data/aproxy.db"
export APROXY_SERVER_AUTH_TOKEN="my-secret-token"
./aproxy

# Use authenticated proxy with curl
curl -x http://localhost:8080 \
  -H "Proxy-Authorization: Bearer my-secret-token" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArnabXD/aproxy](https://github.com/ArnabXD/aproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
