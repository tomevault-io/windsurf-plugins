---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

tosage is a Go application that tracks token usage from Claude Code, Cursor, AWS Bedrock, and Google Vertex AI, and sends metrics to Prometheus. It can run in CLI mode (outputs today's token count) or daemon mode (system tray application with periodic metrics sending).

## Key Commands

### Build
```bash
# Build for current platform (macOS)
make build

# Build for macOS ARM64
make build-darwin

# Build all architectures
make build-all

# Build app bundle for macOS
make app-bundle-arm64

# Build DMG installer (unsigned)
make dmg-arm64

# Build signed DMG (requires CODESIGN_IDENTITY)
make dmg-signed-arm64

# Build signed and notarized DMG
# Requires: CODESIGN_IDENTITY, API_KEY_ID, API_KEY_PATH, API_ISSUER
make dmg-notarized-arm64
```

### Test
```bash
# Run all tests
make test

# Run tests with coverage
make test-coverage

# Run integration tests
make test-integration
```

### Code Quality
```bash
# Run linter (golangci-lint)
make lint

# Format code
make fmt

# Run go vet
make vet

# Run all checks (fmt, vet, lint, test)
make check
```

### Development
```bash
# Run in CLI mode
make run-cli

# Run in daemon mode
make run-daemon

# Install as system daemon
make install-daemon

# Uninstall system daemon
make uninstall-daemon

# Download dependencies
make deps

# Tidy dependencies
make tidy

# Clean build artifacts
make clean

# Deep clean including caches
make clean-all
```

## Architecture

The project follows Clean Architecture with clear separation of concerns:

### Domain Layer (`domain/`)
- **entity/**: Core business entities
  - `cc_entry.go`: Claude Code usage entry
  - `cursor_usage.go`: Cursor usage data (premium requests, usage-based pricing)
  - `vertex_ai_usage.go`: Google Cloud Vertex AI usage data
  - `bedrock_usage.go`: AWS Bedrock usage data
  - `metric_record.go`: Generic metric record for export operations
- **repository/**: Repository interfaces
  - `cc_repository.go`: Claude Code data repository interface
  - `cursor_api_repository.go`: Cursor API interface
  - `vertex_ai_repository.go`: Vertex AI metrics interface
  - `bedrock_repository.go`: Bedrock CloudWatch metrics interface
  - `metrics_repository.go`: Prometheus metrics sender interface
  - `csv_export_repository.go`: CSV export interface
  - `config_repository.go`: Configuration persistence interface
  - `timezone_repository.go`: Timezone handling interface
- **errors.go**: Domain-specific errors
- **logger.go**: Logger interface
- **logger_factory.go**: Logger factory for creating logger instances

### Infrastructure Layer (`infrastructure/`)
- **auth/**: Authentication implementations
  - `vertex_ai_auth.go`: Vertex AI service account authentication
- **config/**: Application configuration
  - `app_config.go`: Main configuration structure with environment variable support
- **di/**: Dependency injection container
  - `container.go`: Main DI container
  - `container_darwin.go`: macOS-specific implementations
  - `cli_controller.go`: CLI mode controller factory
- **logging/**: Logger implementations (debug, promtail)
- **repository/**: Repository implementations
  - `cursor_api_repository.go`: Cursor API client for usage data
  - `cursor_db_repository.go`: SQLite database for Cursor token history
  - `jsonl_cc_repository.go`: Claude Code data file reader
  - `prometheus_metrics_repository.go`: Prometheus remote write
  - `vertex_ai_rest_repository.go`: Vertex AI REST API client
  - `vertex_ai_monitoring_repository.go`: Vertex AI Cloud Monitoring API client
  - `bedrock_cloudwatch_repository.go`: AWS CloudWatch client for Bedrock metrics
  - `csv_writer_repository.go`: CSV file writer for metrics export
  - `json_config_repository.go`: JSON configuration file handler
  - `noop_metrics_repository.go`: No-op implementation for testing
  - `remote_write_client.go`: Prometheus remote write HTTP client
- **service/**: Infrastructure services
  - `timezone_service_impl.go`: Timezone handling implementation

### Use Case Layer (`usecase/`)
- **interface/**: Service interfaces
  - `cc_service.go`: Claude Code service interface
  - `cursor_service.go`: Cursor service interface
  - `vertex_ai_service.go`: Vertex AI service interface
  - `bedrock_service.go`: Bedrock service interface
  - `metrics_service.go`: Metrics aggregation interface
  - `csv_export_service.go`: CSV export interface
  - `config_service.go`: Configuration management interface
  - `config_migration_service.go`: Configuration migration interface
  - `restart_manager.go`: Application restart management interface
  - `status_service.go`: Application status interface
- **impl/**: Service implementations
  - `cc_service_impl.go`: Claude Code data processing
  - `cursor_service_impl.go`: Cursor API integration and token tracking
  - `vertex_ai_service_impl.go`: Vertex AI metrics collection
  - `bedrock_service_impl.go`: Bedrock metrics collection
  - `metrics_service_impl.go`: Metrics collection and sending
  - `metrics_data_collector_impl.go`: Unified metrics data collection
  - `csv_export_service_impl.go`: CSV export functionality
  - `config_service_impl.go`: Configuration management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ca-srg/tosage](https://github.com/ca-srg/tosage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
