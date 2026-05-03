---
trigger: always_on
description: A Go application that fetches DMARC reports from IMAP mailboxes, parses them, and displays them in a Vue.js dashboard. Also supports MCP (Model Context Protocol) for AI assistant integration.
---

# Parse DMARC - Project Guide

A Go application that fetches DMARC reports from IMAP mailboxes, parses them, and displays them in a Vue.js dashboard. Also supports MCP (Model Context Protocol) for AI assistant integration.

## Tech Stack

- **Backend**: Go 1.25.4 (see go.mod for exact version)
- **Frontend**: Vue.js 3 with Vite
- **Database**: SQLite (supports both CGO and pure-Go variants)
- **Package Manager**: Bun (for frontend)
- **Task Runner**: Just (Justfile)
- **CLI Framework**: urfave/cli/v3
- **JSON Library**: goccy/go-json (high-performance)
- **Logging**: rs/zerolog (structured logging)
- **Metrics**: Prometheus client_golang
- **MCP SDK**: modelcontextprotocol/go-sdk

## Project Structure

```
parse-dmarc/
├── main.go                    # Main application entry point
├── internal/
│   ├── api/                   # REST API server and embedded frontend
│   │   └── server.go          # HTTP server, routes, metrics middleware
│   ├── config/                # Configuration management (JSON + env vars)
│   │   └── config.go          # Config loading and validation
│   ├── imap/                  # IMAP client for fetching emails
│   │   └── client.go          # Email fetching logic
│   ├── logger/                # Structured logging setup
│   │   └── logger.go          # Zerolog configuration
│   ├── mcp/                   # MCP (Model Context Protocol) server
│   │   ├── server.go          # MCP server (stdio and HTTP/SSE)
│   │   ├── tools.go           # MCP tool implementations
│   │   └── oauth/             # OAuth2 authentication for MCP
│   │       ├── config.go      # OAuth configuration
│   │       ├── middleware.go  # Bearer auth middleware
│   │       ├── metadata.go    # RFC 9728 metadata endpoint
│   │       └── verifier.go    # Token verification
│   ├── metrics/               # Prometheus metrics
│   │   └── metrics.go         # Metrics definitions and HTTP middleware
│   ├── parser/                # DMARC XML parser
│   │   ├── dmarc.go           # Parsing logic
│   │   └── dmarc_test.go      # Parser tests
│   └── storage/               # SQLite database layer
│       ├── common.go          # Shared SQL queries and types
│       ├── sqlite_cgo.go      # CGO SQLite (mattn/go-sqlite3)
│       └── sqlite_no_cgo.go   # Pure Go SQLite (modernc.org/sqlite)
├── src/                       # Vue.js 3 frontend source
│   ├── App.vue                # Main application component
│   ├── main.js                # Vue entry point
│   ├── assets/
│   │   └── base.css           # Base styles
│   ├── stores/                # Pinia-like state management
│   │   ├── index.js           # Store exports
│   │   ├── theme.js           # Theme (dark/light/system) store
│   │   └── settings.js        # API endpoint settings store
│   └── components/
│       ├── dashboard/
│       │   ├── DashboardHero.vue   # Dashboard header/hero section
│       │   ├── RecentReports.vue   # Recent reports list
│       │   └── ReportDrawer.vue    # Report detail drawer
│       ├── settings/
│       │   └── SettingsModal.vue   # Settings modal (theme, API endpoint)
│       └── tools/
│           └── DnsGenerator.vue    # DMARC DNS record generator
├── public/                    # Static frontend assets (favicons, logos)
├── assets/                    # Project assets (screenshots, images)
├── deploy/                    # Deployment configurations
│   ├── coolify.yaml           # Coolify deployment
│   ├── captain-definition     # CapRover deployment
│   ├── digitalocean/          # DigitalOcean Droplet/Marketplace
│   │   ├── packer.pkr.hcl     # Packer image build
│   │   ├── marketplace.yaml   # DO Marketplace metadata
│   │   └── scripts/           # Setup scripts
│   └── dokploy/               # Dokploy deployment
│       ├── template.toml
│       └── docker-compose.yml
├── grafana/                   # Grafana monitoring
│   ├── dashboard.json         # Pre-built dashboard
│   └── provisioning.yaml      # Auto-provisioning config
├── scripts/                   # Utility scripts
├── Justfile                   # Build commands
├── Dockerfile                 # Multi-stage Docker build
├── compose.yml                # Docker Compose for local dev
├── parse-dmarc.service        # systemd service file
├── zeabur.yml                 # Zeabur deployment template
├── render.yaml                # Render deployment config
├── Northflank.json            # Northflank deployment config
├── ROADMAP.md                 # Product roadmap
├── CONTRIBUTING.md            # Contribution guidelines
├── .goreleaser.yml            # Release automation
└── .github/workflows/ci.yml   # CI/CD pipeline
```

## Development Commands

```bash
# Install all dependencies (Go + Node)
just install-deps

# Build full application (frontend + backend)
just build

# Build with CGO (for native SQLite)
just build-cgo

# Run development server with hot reload (uses air)
just dev

# Run frontend dev server only
just frontend-dev

# Run tests
just test

# Generate config file
just config

# Clean build artifacts
just clean

# Install binary to /usr/local/bin
just install

# Update Zeabur template
just update-zeabur-template
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmarcguardhq/dmarcguard](https://github.com/dmarcguardhq/dmarcguard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
