---
trigger: always_on
description: This document provides guidance for GitHub Copilot when working with the go-sqlcmd repository.
---

# GitHub Copilot Instructions for go-sqlcmd

This document provides guidance for GitHub Copilot when working with the go-sqlcmd repository.

## Project Overview

go-sqlcmd is a Go-based command line tool (`sqlcmd`) for working with Microsoft SQL Server, Azure SQL Database, and Azure Synapse. The project aims to be a complete port of the original ODBC sqlcmd to Go, utilizing the [go-mssqldb](https://github.com/microsoft/go-mssqldb) driver.

## Repository Structure

```
/
├── cmd/                    # Entry points for the application
│   ├── modern/             # Modern CLI entry point (Cobra-based)
│   │   ├── root/           # Root command and subcommands
│   │   └── sqlconfig/      # SQL configuration management
│   └── sqlcmd/             # Legacy CLI entry point (Kong-based)
├── pkg/                    # Public packages consumable by other hosts
│   └── sqlcmd/             # Core sqlcmd functionality
├── internal/               # Internal packages (not for external use)
│   ├── buffer/             # Buffer management
│   ├── cmdparser/          # Command parsing utilities
│   ├── color/              # Console coloring
│   ├── config/             # Configuration management
│   ├── container/          # Docker/Podman container management
│   ├── credman/            # Credential management
│   ├── http/               # HTTP utilities
│   ├── io/                 # I/O utilities
│   ├── localizer/          # Localization support
│   ├── net/                # Network utilities
│   ├── output/             # Output formatting
│   ├── pal/                # Platform abstraction layer
│   ├── secret/             # Secret/credential management
│   ├── sql/                # SQL-related utilities
│   ├── test/               # Test utilities
│   ├── tools/              # Development tools
│   └── translations/       # Localized string translations
├── build/                  # Build scripts and templates
├── testdata/               # Test data files
├── release/                # Release-related files
└── .github/                # GitHub workflows and configurations
```

## Building the Project

### Build Commands

```bash
# Build the sqlcmd executable
./build/build.sh       # Linux/macOS
.\build\build.cmd      # Windows

# Or build directly with Go
go build -o sqlcmd ./cmd/modern
```

### Dependencies

The project uses Go modules. Run `go mod download` to fetch dependencies.

## Testing

### Running Tests

```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run tests for a specific package
go test -v ./pkg/sqlcmd/...
go test -v ./internal/config/...
```

### Test Environment Variables

Tests may require the following environment variables for database connectivity:
- `SQLCMDSERVER` - SQL Server hostname
- `SQLCMDPORT` - SQL Server port (default: 1433)
- `SQLCMDUSER` - Username (e.g., `sa`)
- `SQLCMDPASSWORD` - Password
- `SQLCMDDATABASE` - Database name

## Code Style and Conventions

### Go Standards

- Follow standard Go conventions and idioms
- Use `gofmt` for formatting
- Use tabs for indentation (as specified in `.editorconfig`)
- Follow effective Go guidelines: https://go.dev/doc/effective_go

### File Headers

All Go files should include the following copyright header:
```go
// Copyright (c) Microsoft Corporation.
// Licensed under the MIT license.
```

### Package Documentation

Each package should have a `doc.go` file with package-level documentation.

### Error Handling

- Use the standard Go error handling patterns
- For user-facing errors, prefer localized error messages via the `internal/localizer` package
- Wrap errors with context when propagating

### Naming Conventions

- Use camelCase for unexported identifiers
- Use PascalCase for exported identifiers
- Acronyms should be all uppercase (e.g., `SQL`, `URL`, `HTTP`)
- Package names should be lowercase, single-word if possible

## CLI Architecture

### Modern CLI (Cobra-based)

The modern CLI is located in `cmd/modern/` and uses the [Cobra](https://github.com/spf13/cobra) library. Key points:
- Root command is in `cmd/modern/root.go`
- Subcommands are in `cmd/modern/root/` directory
- Uses dependency injection for testability

### Legacy CLI (Kong-based)

The legacy CLI is in `cmd/sqlcmd/` and maintains backward compatibility with the original ODBC sqlcmd.

### Command Structure

When adding new commands:
1. Create the command in `cmd/modern/root/`
2. Follow the existing pattern for subcommands (see `query.go`, `start.go`, `stop.go`)
3. Add corresponding tests with `_test.go` suffix

## Configuration

- Configuration files are stored in `~/.sqlcmd/sqlconfig`
- Use the `internal/config` package for configuration management
- Viper is used for configuration parsing (`internal/config/viper.go`)

## Container Support

The project supports creating SQL Server instances using Docker or Podman:
- Container management is in `internal/container/`
- Supports SQL Server images

## Localization

- Localized strings are in `internal/translations/`
- Use the `internal/localizer` package for localized messages
- Supported languages: Chinese (Simplified/Traditional), English, French, German, Italian, Japanese, Korean, Portuguese (Brazil), Russian, Spanish

### Adding Localizable Strings


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/go-sqlcmd](https://github.com/microsoft/go-sqlcmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
