---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ts-ssh is a simplified Go-based SSH and SCP client that uses Tailscale's `tsnet` library to provide userspace connectivity to Tailscale networks without requiring a full Tailscale daemon. The project enables secure SSH connections and file transfers over a Tailnet with enterprise-grade security and a minimal, ssh-like CLI interface.

**Design Philosophy**: Simplicity over features. This tool mimics the standard `ssh` command with minimal flags and maximum clarity.

## Guidance Notes

- **Quality Score Tracking**: Do not store quality scores in any artifacts, including markdown files, code comments, commit messages, or pull request descriptions. Quality metrics, including security assessments, should be reported back to the project lead but not memorialized in project artifacts.
- **Code Simplicity**: Keep the codebase small and maintainable. Avoid adding complexity unless absolutely necessary.

## CLI Architecture

ts-ssh uses a simple, flag-based CLI that mimics the standard `ssh` command:

### Basic Usage
```bash
ts-ssh [options] [user@]host[:port] [command...]
ts-ssh -scp source dest
```

### Core Features
- **SSH Connection**: Just like `ssh`, connect to any host on your Tailnet
- **SCP Transfer**: Simple file transfer with `-scp` flag
- **SOCKS5 Proxy**: `-D` flag for dynamic port forwarding (VSCode Remote SSH compatible)
- **Port Specification**: Use `-p` flag or `host:port` syntax
- **PTY Control**: `-T` flag to disable pseudo-terminal allocation
- **Verbose Mode**: `-v` for debugging and authentication URLs
- **Flexible Usernames**: Supports dots in usernames (e.g., `first.last`)

### No Subcommands
- No complex subcommand structure
- No dual CLI modes
- No internationalization
- No styling frameworks
- Just simple, direct flags

## Release Considerations

- Ensure the `--version` flag works correctly during release builds
- Verify proper version flag implementation when cross-compiling for different platforms

## Common Commands

### Build
```bash
go build -o ts-ssh .
```

### Run Tests
```bash
# Run all tests
go test ./...

# Run with verbose output
go test ./... -v

# Run with coverage
go test ./... -cover

# Run security tests
go test ./... -run "Test.*[Ss]ecure" -v

# Check for race conditions
go test ./... -race
```

### Cross-compile Examples
```bash
# Windows AMD64
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o ts-ssh-windows.exe .

# macOS ARM64
CGO_ENABLED=0 GOOS=darwin GOARCH=arm64 go build -o ts-ssh-darwin-arm64 .

# Linux AMD64
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ts-ssh-linux-amd64 .
```

### Run Application

```bash
# Basic SSH connection
./ts-ssh hostname
./ts-ssh user@hostname
./ts-ssh user@hostname:2222

# Execute remote command
./ts-ssh hostname uptime
./ts-ssh user@hostname "ls -la /tmp"

# SCP file transfer
./ts-ssh -scp file.txt hostname:/tmp/
./ts-ssh -scp hostname:/tmp/file.txt ./downloads/

# SOCKS5 dynamic port forwarding
./ts-ssh -D 1080 hostname            # SOCKS proxy on localhost:1080
./ts-ssh -D 0.0.0.0:1080 hostname    # Bind to all interfaces (with warning)

# With options
./ts-ssh -v hostname                  # Verbose mode
./ts-ssh -p 2222 hostname            # Custom port
./ts-ssh -l alice hostname           # Specify username (supports dots: first.last)
./ts-ssh -i ~/.ssh/custom_key hostname  # Custom key
./ts-ssh -T hostname "cat /etc/hostname"  # Disable PTY allocation

# Get help
./ts-ssh --help
./ts-ssh --version
```

## Key Dependencies

### Core Libraries (Minimal Set)
- **Tailscale**: Core networking and `tsnet` integration
- **golang.org/x/crypto/ssh**: SSH client implementation
- **golang.org/x/term**: Terminal handling for interactive sessions
- **github.com/bramvdbogaerde/go-scp**: SCP file transfer

### Removed Dependencies
The following were removed to simplify the codebase:
- ❌ Charmbracelet Fang, Lipgloss, Huh (UI frameworks)
- ❌ Spf13 Cobra (command framework)
- ❌ Internationalization (i18n) system
- ❌ Complex CLI modes

## Code Structure

```
ts-ssh/
├── main.go              # ~700 lines - main CLI logic + SOCKS5
├── constants.go         # ~52 lines - constants
├── main_test.go         # ~850 lines - tests (including SOCKS5 tests)
└── internal/
    ├── client/
    │   ├── scp/         # SCP client implementation
    │   └── ssh/         # SSH client implementation
    ├── config/          # Configuration constants
    ├── crypto/pqc/      # Post-quantum cryptography
    ├── errors/          # Error handling
    ├── platform/        # Platform-specific code
    └── security/        # Security validation
```

**Total**: ~5,250 lines (including SOCKS5 proxy support and comprehensive tests)

## Code Quality Standards

### Linting and Formatting
```bash
# Format code (required before commits)
go fmt ./...

# Run linter (should show no issues)
golangci-lint run

# Vet for potential issues
go vet ./...
```

### Test Coverage Expectations
- **Error handling**: Target 80%+ coverage
- **Security modules**: 100% coverage required
- **Core functionality**: 70%+ coverage minimum
- **Main CLI**: Test all parsing functions

## Architecture Insights


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derekg/ts-ssh](https://github.com/derekg/ts-ssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
