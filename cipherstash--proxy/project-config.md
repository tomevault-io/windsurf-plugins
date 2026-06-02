---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CipherStash Proxy is a PostgreSQL proxy that provides **transparent, searchable encryption** for existing applications. It sits between applications and PostgreSQL databases, automatically encrypting sensitive data while preserving the ability to query encrypted values using equality, comparison, and ordering operations.

Key capabilities:
- Zero-change SQL queries - applications connect to Proxy instead of directly to PostgreSQL
- EQL v2 (Encrypt Query Language) for searchable encryption using CipherStash ZeroKMS
- Support for encrypted equality, comparison, ordering, and grouping operations
- Written in Rust for performance with strongly-typed SQL statement mapping

## Architecture

### High-Level Components

**Core Proxy (`packages/cipherstash-proxy/`):**
- `postgresql/` - PostgreSQL wire protocol implementation, message parsing, and client handling
- `encrypt/` - Integration with CipherStash ZeroKMS for key management and encryption operations
- `config/` - Configuration management for database connections, TLS, and encryption settings
- `eql/` - EQL v2 types and encryption abstractions

**EQL Mapper (`packages/eql-mapper/`):**
- SQL parsing and type inference engine
- Transformation rules for converting plaintext SQL to encrypted operations
- Schema analysis and column mapping for encryption

**Integration Tests (`packages/cipherstash-proxy-integration/`):**
- Comprehensive test suite covering encryption scenarios
- Language-specific integration tests (Python, Go, Elixir)

**Showcase (`packages/showcase/`):**
- Healthcare data model demonstrating EQL v2 encryption
- Example of realistic encrypted application with foreign keys and relationships

### Request Flow

1. Application connects to Proxy (port 6432) using standard PostgreSQL protocol
2. Proxy intercepts SQL statements and uses EQL Mapper to analyze query structure
3. For encrypted columns, Proxy transforms SQL using EQL v2 operations
4. Encrypted queries are sent to actual PostgreSQL database
5. Results are decrypted before returning to application

## Development Commands

### Prerequisites Setup
```bash
# Install mise (required for all development)
brew install mise  # macOS
mise trust --yes && mise install

# Start PostgreSQL containers
mise run postgres:up --extra-args "--detach --wait"
mise run postgres:setup  # Install EQL and schema
```

> **macOS Note:** If you hit file descriptor limits during development (e.g. "Too many open files"), you may need to increase the limit:
> ```bash
> ulimit -n 10240
> ```
> To make this persistent, add it to your shell profile (e.g. `~/.zshrc`).

### Core Development Workflow
```bash
# Build and run Proxy as a process (development)
mise run proxy

# Run Proxy in container (integration testing)
mise run proxy:up --extra-args "--detach --wait"

# Kill running processes
mise run proxy:kill

# Reset database state
mise run reset
```

### Testing
```bash
# Full test suite (hygiene + unit + integration)
mise run test

# Hygiene checks only
mise run check

# Unit tests only
mise run test:unit [test_name]

# Integration tests
mise run test:integration

# Language-specific integration tests
mise run test:integration:lang:python
mise run test:integration:lang:golang

# Individual test packages
mise run test:local:integration  # cipherstash-proxy-integration
mise run test:local:mapper       # eql-mapper
```

### Database Management
```bash
# Connect to Proxy interactively
mise run proxy:psql

# Connect directly to PostgreSQL (bypassing Proxy)
mise run postgres:psql

# Clean shutdown
mise run postgres:down
```

## Configuration

### Authentication & Encryption
Proxy requires CipherStash credentials configured in `mise.local.toml`:
```toml
CS_WORKSPACE_CRN = "crn:region:workspace-id"
CS_CLIENT_ACCESS_KEY = "your-access-key"
CS_DEFAULT_KEYSET_ID = "your-keyset-id"
CS_CLIENT_ID = "your-client-id"
CS_CLIENT_KEY = "your-client-key"
```

### PostgreSQL Port Conventions
- `5532` - PostgreSQL latest (non-TLS)
- `5617` - PostgreSQL 17 (TLS)
- `6432` - CipherStash Proxy

Container names: `postgres`, `postgres-tls`, `proxy`, `proxy-tls`

### Logging Configuration
Set granular log levels by target:
```bash
CS_LOG__MAPPER_LEVEL=debug
CS_LOG__AUTHENTICATION_LEVEL=debug
CS_LOG__ENCRYPT_LEVEL=debug
CS_LOG__ZEROKMS_LEVEL=debug
```

Available targets: `DEVELOPMENT`, `AUTHENTICATION`, `CONFIG`, `CONTEXT`, `ENCODING`, `ENCRYPT`, `DECRYPT`, `ENCRYPT_CONFIG`, `ZEROKMS`, `MIGRATE`, `PROTOCOL`, `MAPPER`, `SCHEMA`, `SLOW_STATEMENTS`

## Key Development Patterns

### Error Handling
- All errors defined in `packages/cipherstash-proxy/src/error.rs`
- Errors grouped by problem domain (not module structure)
- Customer-facing errors include friendly messages and documentation links
- Use descriptive variant names without "Error" suffix

### Testing Patterns
- Use `unwrap()` instead of `expect()` unless providing meaningful context
- Prefer `assert_eq!` over `assert!` for equality checks
- Integration tests use Docker containers for reproducible environments

### SQL Transformation
- EQL Mapper handles SQL parsing and type inference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cipherstash/proxy](https://github.com/cipherstash/proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
