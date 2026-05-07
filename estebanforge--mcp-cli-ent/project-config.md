---
trigger: always_on
description: MCP CLI-Ent is a Go-based standalone CLI client for MCP (Model Context Protocol) servers. It enables interaction with MCP servers without loading them into Claude Code's context window, providing intelligent context management for AI agents.
---

# Project Overview

MCP CLI-Ent is a Go-based standalone CLI client for MCP (Model Context Protocol) servers. It enables interaction with MCP servers without loading them into Claude Code's context window, providing intelligent context management for AI agents.

**Key Characteristics:**
- **Language**: Go 1.21+
- **Purpose**: Standalone MCP server interaction with minimal context pollution and persistent daemon support
- **Architecture**: Transport-agnostic JSON-RPC 2.0 implementation with cross-platform daemon service
- **Compatibility**: Works with existing Claude Code/VSCode `mcp_servers.json` configurations
- **Distribution**: Single binary with zero runtime dependencies
- **Session Persistence**: Gemini CLI-like persistent browser sessions for automation workflows

## Repository Structure

```
mcp-cli-ent/
├── cmd/mcp-cli-ent/           # Main CLI entry point
│   └── main.go               # Application bootstrap
├── internal/
│   ├── client/               # MCP client transport implementations
│   │   ├── interface.go      # Client factory and interfaces
│   │   ├── http.go           # HTTP transport client
│   │   └── stdio.go          # Stdio transport client
│   ├── cli/                  # CLI command definitions
│   │   ├── root.go           # Root command and global flags
│   │   └── commands.go       # All subcommands
│   ├── config/               # Configuration management
│   │   ├── config.go         # Configuration loading and validation
│   │   └── types.go          # Configuration type definitions
│   ├── daemon/               # Persistent daemon service
│   │   ├── daemon.go         # Main daemon implementation
│   │   ├── types.go          # Daemon-specific types
│   │   ├── manager.go        # Cross-platform daemon lifecycle
│   │   ├── client.go         # Smart CLI-daemon bridge
│   │   ├── server.go         # HTTP API handlers
│   │   ├── endpoint.go       # Cross-platform endpoint management
│   │   └── platform.go       # Platform-specific process management
│   └── mcp/                  # MCP protocol implementation
│       ├── protocol.go       # Client interface and validation
│       └── types.go          # JSON-RPC 2.0 protocol types
├── pkg/version/              # Version information
│   └── version.go            # Build-time version variables
├── scripts/                  # Installation and utility scripts
│   ├── install.sh            # Unix/Linux/macOS installer
│   └── test-*.sh             # Installer testing scripts
├── .github/workflows/        # CI/CD pipelines
│   ├── build.yml             # Test and build workflow
│   └── release.yml           # Release automation
├── go.mod                    # Go module definition
├── go.sum                    # Go dependency checksums
├── Makefile                  # Build automation
├── VERSION                   # Release version file
├── mcp_servers.example.json  # Example MCP server configuration
├── README.md                 # Project documentation
├── CHANGELOG.md              # Version history
└── LICENSE                   # MIT License
```

## Development Environment Setup

### Prerequisites
- Go 1.21 or later
- Git
- Make

### Initial Setup
```bash
# Clone repository
git clone https://github.com/EstebanForge/mcp-cli-ent.git
cd mcp-cli-ent

# Install development tools
make dev-setup

# Download dependencies
make deps

# Build for current platform
make build

# Run tests
make test
```

## Build System

### Make Targets

**Development:**
```bash
make build          # Build unsigned local binary (bin/mcp-cli-ent)
make sign           # Sign local binary on macOS (optional)
make build-signed   # Build + sign local binary on macOS (optional)
make build-all      # Build for all platforms (dev version)
make test           # Run tests (deps + verify + go test)
make test-coverage  # Run tests with coverage report
make fmt            # Format code (go fmt + optional goimports)
make vet            # Run go vet
make lint           # Run golangci-lint
make check          # Full flow: fmt + vet + lint + test + build (+check-config)
make ci             # CI alias for make check
make clean          # Clean build artifacts
make deps           # Download and tidy dependencies
```

**Release:**
```bash
make build-release  # Build for all platforms (release version)
make release        # Full release build (test + lint + build-release)
make release-sign   # Sign release macOS binaries (optional)
make notarize-release # Notarize release artifacts (optional)
make set-version VERSION=1.2.3  # Set release version
```

**Utility:**
```bash
make dev-setup      # Install development tools
make run-example    # Build and test with example config
make install        # Install to GOPATH/bin
make help           # Show all available targets
```

### Version Management
- **Development Version**: Generated from `git describe --tags --always --dirty`
- **Release Version**: Read from `VERSION` file
- **Policy**: Do not edit `VERSION` manually; release workflow sets it from the tag.
- **Build Metadata**: Commit hash, build date, and Go version embedded at build time


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EstebanForge/mcp-cli-ent](https://github.com/EstebanForge/mcp-cli-ent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
