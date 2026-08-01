---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

grant-rs is a CLI tool written in Rust for managing PostgreSQL/Redshift database roles and privileges in GitOps style. It allows declarative configuration of database permissions through YAML files.

**Status**: Production-ready with comprehensive security, safety features, and full user lifecycle management.

## Development Commands

### Building and Testing
```bash
# Build the project
cargo build

# Run tests (requires PostgreSQL running)
cargo test

# Run with logging
RUST_LOG=debug cargo run

# Install locally
cargo install --path .
```

### Database Setup for Testing
```bash
# Start PostgreSQL via Docker
docker-compose up -d

# Connect to test database (required for tests)
# Connection: postgres://postgres:postgres@localhost:5432/postgres
```

### Running the CLI Tool
```bash
# Generate sample configuration
cargo run -- gen --target ./cluster

# Validate configuration
cargo run -- validate -f examples/example.yaml

# Preview changes (dry-run mode)
cargo run -- apply -f examples/example.yaml --dryrun

# Apply configuration to database
cargo run -- apply -f examples/example.yaml

# Apply with user cleanup (GitOps mode - destructive!)
cargo run -- apply -f examples/example.yaml --delete-users

# Apply all configs in directory recursively
cargo run -- apply -f ./configs --all

# Inspect current database state
cargo run -- inspect -f examples/example.yaml

# Generate random password
cargo run -- gen-pass --user username
```

## Architecture

### Core Modules
- **config/**: Configuration parsing and validation
  - `connection.rs` - Database connection configuration
  - `role_*.rs` - Role definitions (database, schema, table levels)
  - `user.rs` - User management and password handling
  - `sql_utils.rs` - **NEW**: Shared SQL escaping utilities (security-critical)
  - `config_base.rs` - Main configuration structure
- **apply.rs** - Main logic for applying privilege changes to database
  - **NEW**: Comprehensive module documentation
  - **NEW**: User deletion with safety features
  - Idempotent privilege grants
- **inspect.rs** - Database introspection and current state analysis
- **validate.rs** - Configuration file validation with proper error reporting
- **gen.rs** - Configuration generation utilities
- **connection.rs** - Database connection management
- **cli.rs** - Command-line interface definitions

### Key Design Patterns
- **Configuration-driven**: All database changes defined in YAML files
- **Three privilege levels**: DATABASE, SCHEMA, TABLE with inheritance
- **GitOps approach**: Version-controlled privilege management
- **Idempotent operations**: Safe to run multiple times
- **Include/exclude patterns**: Support for + and - prefixes in table grants
- **Security-first**: All SQL identifiers properly escaped via shared `sql_utils` module
- **DRY principle**: Single source of truth for SQL escaping logic
- **Safety by design**: Opt-in destructive operations, superuser protection
- **Honest types**: Function signatures accurately reflect behavior (no unnecessary Result types)

### Dependencies
- `structopt` for CLI parsing
- `postgres` for database connectivity
- `serde`/`serde_yaml` for configuration parsing
- `anyhow` for error handling
- `log`/`env_logger` for logging

## Testing Strategy

Tests require a running PostgreSQL instance. The project includes:
- **Unit tests**: Configuration parsing, validation, SQL generation
- **Security tests**: SQL injection prevention in `sql_utils.rs`
- **Integration tests**: Database operations with actual PostgreSQL
- **Example configurations**: In `examples/` directory
- **Development database**: Via `docker-compose.yaml`

**Test Coverage**:
- Configuration parsing: 95%+ ✅
- SQL generation: 90%+ ✅
- Security (SQL injection): Comprehensive ✅
- CLI interface: 70%+ ✅

**Running Tests**:
```bash
# All tests (requires PostgreSQL)
cargo test

# Specific test
cargo test test_escape_identifier

# With output
cargo test -- --nocapture
```

## Configuration Format

The tool uses YAML configuration with three main sections:
- `connection`: Database connection details (supports environment variables)
- `roles`: Privilege definitions at database/schema/table levels
- `users`: User accounts and role assignments

Supports password encryption (MD5) and environment variable substitution in connection strings.

## Security & Safety Features

### SQL Injection Prevention
- **Shared SQL utilities** (`config/sql_utils.rs`): Single source of truth for escaping
  - `escape_identifier()`: For table/column/role names (double-quote escaping)
  - `escape_sql_string()`: For string literals (single-quote escaping)
- **Comprehensive testing**: All SQL generation tested for injection vulnerabilities
- **Used consistently**: All modules use shared utilities (no local implementations)

### User Deletion Safety
- **Opt-in only**: Requires explicit `--delete-users` flag
- **Superuser protection**: Never deletes superusers automatically
- **Dry-run preview**: See what would be deleted before committing
- **Graceful errors**: One deletion failure doesn't stop entire operation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duyet/grant-rs](https://github.com/duyet/grant-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
