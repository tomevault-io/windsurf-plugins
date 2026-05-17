---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ReactorCA is a Go-based CLI tool for managing a private PKI (Public Key Infrastructure) for homelab and small business environments.

## Architecture

The project follows Clean Architecture principles with clear separation of concerns:

- `cmd/ca/` - CLI commands and entry point
- `internal/app/` - Application service layer (business logic orchestration)
- `internal/domain/` - Core domain interfaces, types, and errors
- `internal/infra/` - Infrastructure implementations (crypto, storage, config loading)
  - `internal/infra/identity/` - Identity provider implementations (password, SSH, age plugins)
- `internal/ui/` - Console output formatting and user interaction

Key architectural principles:
- **Stateless operations**: Each command execution is independent
- **Configuration-driven**: All settings defined in YAML files (`config/ca.yaml`, `config/hosts.yaml`)
- **Secure storage**: Private keys encrypted using age-based encryption with configurable identity providers
- **Clean interfaces**: Domain interfaces implemented by infrastructure layer

## Tree-Sitter Integration

Initialize tree-sitter for code exploration:
```bash
# Configure and register the project
mcp__tree-sitter__configure --config-path .tree-sitter-mcp-config.yaml
mcp__tree-sitter__register_project_tool --path /Users/self/Documents/projects/ReactorCA --name ReactorCA --description "Go-based CLI tool for managing a private PKI"
```

## Development Commands

**IMPORTANT**: If you want to use the `cd` shell command, ALWAYS wrap it like `sh -c "cd dir && ls"`.

This project uses [just](https://github.com/casey/just) for task automation. Run `just help` to see available commands.

### Common Commands
```bash
# Build the binary (debug mode)
just build

# Build optimized release binary
just release

# Format code
just fmt

# Run linting and checks
just lint

# Run tests
just test             # All tests (unit, integration, e2e)
just test unit        # Unit tests only
just test integration # Integration tests only
just test e2e         # End-to-end tests only

# Direct Go test commands with build tags
go test -v ./cmd/... ./internal/...              # Unit tests (default, no tags)
go test -v -tags integration ./test/integration/... # Integration tests
go test -v -tags e2e ./test/e2e/...                 # End-to-end tests
go test -v -tags browser ./test/e2e/... -timeout=10m # Browser compatibility tests

# Full CI pipeline
just ci

# Complete check (lint, build, test, tidy)
just check
```

### CLI Usage Pattern
```bash
# Initialize a new PKI environment
./ca init

# CA Management
./ca ca create                  # Create new CA
./ca ca renew                   # Renew CA certificate
./ca ca rekey                   # Regenerate CA private key
./ca ca info                    # Show CA information
./ca ca import                  # Import external CA
./ca ca export-key              # Export CA private key
./ca ca reencrypt               # Change master password

# Host Certificate Management
./ca host issue web-server      # Issue new certificate
./ca host deploy web-server     # Deploy certificate
./ca host export-key web-server # Export private key
./ca host import-key web-server # Import private key
./ca host sign-csr              # Sign external CSR
./ca host list                  # List all certificates
./ca host clean                 # Remove orphaned certificates

# Configuration
./ca config validate            # Validate configuration files

# Use custom root directory
./ca --root /path/to/pki ca create
```

## Key Components

### Application Layer (`internal/app/application.go`)
Central orchestrator that coordinates between domain interfaces. Key methods:
- `CreateCA()` - Generate new CA key/cert
- `RenewCA()` - Renew CA certificate keeping same key
- `RekeyCA()` - Generate new CA key and certificate
- `InfoCA()` - Display CA certificate information
- `ImportCA()` - Import external CA certificate and key
- `ExportCAKey()` - Export CA private key
- `IssueHost()` - Create/renew host certificates
- `DeployHost()` - Execute deployment commands with variable substitution
- `ExportHostKey()` - Export host private key
- `ImportHostKey()` - Import host private key
- `SignCSR()` - Sign external certificate signing request
- `ReencryptKeys()` - Re-encrypt all keys with new identity provider (with .bak file backup/rollback)
- `ValidateConfig()` - Validate configuration files

### Domain Interfaces (`internal/domain/interfaces.go`)
- `CryptoService` - All cryptographic operations
- `CryptoServiceFactory` - Factory for creating crypto services
- `Store` - Certificate/key persistence 
- `ConfigLoader` - YAML configuration loading
- `IdentityProvider` - Age identity management for encryption/decryption
- `IdentityProviderFactory` - Factory for creating identity providers
- `Commander` - External command execution
- `ValidationService` - Round-trip validation of crypto operations

### Configuration Types (`internal/domain/config.go`)
- `CAConfig` - Root CA settings with encryption configuration
- `HostsConfig` - Host certificate definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serpent213/reactor-ca](https://github.com/serpent213/reactor-ca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
