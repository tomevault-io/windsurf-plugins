---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**grafanactl** is a command-line tool for managing Grafana resources through the REST API. It supports Grafana 12 and above, enabling users to authenticate, manage multiple environments, and perform administrative tasks from the terminal. The tool is particularly useful for dashboards-as-code workflows and CI/CD automation.

## Development Environment

This project uses **devbox** for consistent development environments:

```bash
# Enter devbox shell (includes all required tools)
devbox shell

# Run one-off commands within devbox
devbox run go version

# Add packages
devbox add go@1.24
```

## Common Commands

### Build and Test

```bash
# Build the binary to bin/grafanactl
make build

# Run all tests
make tests

# Run tests for a specific package
go test -v ./internal/config

# Install to $GOPATH/bin
make install

# Run linter
make lint

# Run all checks (lint, tests, build, docs)
make all
```

### Documentation

```bash
# Generate and build all documentation
make docs

# Generate CLI reference documentation
make cli-reference

# Generate environment variable reference
make env-var-reference

# Generate configuration file reference
make config-reference

# Serve documentation locally with live reload
make serve-docs

# Check for drift in generated documentation
make reference-drift
```

### Dependency Management

```bash
# Vendor dependencies
make deps

# Clean build artifacts and dependencies
make clean
```

## Architecture

### Command Structure

grafanactl follows the Cobra command pattern with two main command groups:

1. **config**: Manage configuration contexts for connecting to Grafana instances
   - `config set`: Set configuration values
   - `config unset`: Unset configuration values
   - `config use-context`: Switch between configured contexts
   - `config list-contexts`: List all configured contexts
   - `config current-context`: Show the current context
   - `config view`: View the current configuration
   - `config check`: Validate the configuration

2. **resources**: Manipulate Grafana resources (dashboards, folders, etc.)
   - `resources get`: Get resources from Grafana
   - `resources list`: List resources
   - `resources pull`: Pull resources from Grafana to local files
   - `resources push`: Push local resources to Grafana
   - `resources delete`: Delete resources from Grafana
   - `resources edit`: Edit resources interactively
   - `resources validate`: Validate resource manifests
   - `resources serve`: Serve resources locally with live reload

### Core Packages

**cmd/grafanactl/** - CLI command implementations
- `root/`: Root command setup with logging and flags
- `config/`: Configuration management commands
- `resources/`: Resource manipulation commands
- `fail/`: Error handling and detailed error messages
- `io/`: Output formatting and user messages

**internal/config/** - Configuration management
- Context-based configuration (similar to kubectl contexts)
- Support for multiple Grafana instances (contexts)
- Authentication: basic auth, API tokens
- Environment variable overrides (GRAFANA_SERVER, GRAFANA_TOKEN, etc.)
- Automatic Stack ID discovery for Grafana Cloud
- TLS configuration support

**internal/resources/** - Resource abstraction layer
- `Resource`: Wraps Kubernetes-style unstructured objects for Grafana resources
- `Resources`: Collection with filtering, grouping, and concurrent operations
- Uses `k8s.io/apimachinery` for resource representation
- Supports multiple source formats (JSON, YAML)

**internal/resources/local/** - Local file operations
- `reader.go`: Load resources from disk (supports directories and single files)
- `writer.go`: Save resources to disk with proper formatting

**internal/resources/remote/** - Remote Grafana operations
- `puller.go`: Fetch resources from Grafana API
- `pusher.go`: Upload resources to Grafana API (create/update)
- `deleter.go`: Delete resources from Grafana

**internal/resources/dynamic/** - Dynamic Kubernetes client wrapper
- `namespaced_client.go`: Per-namespace resource operations
- `versioned_client.go`: Version-aware resource client
- Wraps k8s.io/client-go dynamic client

**internal/resources/discovery/** - API discovery
- `registry.go`: Discover available resource types from Grafana API
- `registry_index.go`: Index and cache discovered resources

**internal/resources/process/** - Resource processing
- `managerfields.go`: Handle manager metadata (similar to kubectl's server-side apply)
- `serverfields.go`: Process server-managed fields

**internal/server/** - Local development server (for `resources serve`)
- Chi-based HTTP server with reverse proxy to Grafana
- Live reload via WebSocket
- File watching with fsnotify
- Dashboard and folder preview handlers
- Script execution for generated resources

**internal/grafana/** - Grafana API client
- Wraps grafana-openapi-client-go
- Client construction from context configuration

**internal/format/** - Format detection and conversion
- JSON, YAML format support
- Auto-detection from file extensions

**internal/httputils/** - HTTP utilities
- REST client helpers
- Request/response handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafanactl](https://github.com/grafana/grafanactl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
