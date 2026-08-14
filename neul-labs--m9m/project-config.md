---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code when working with the m9m repository.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code when working with the m9m repository.

## Project Overview

m9m is **the n8n alternative without the bugs — faster, more reliable workflow automation**. It's an open-source workflow automation engine in Go that runs n8n workflow JSON unchanged, executes 5–10× faster, uses 70% lower memory, and ships as a single 30 MB binary with zero runtime dependencies.

The canonical positioning, taglines, and one-liners are listed below. **Use them verbatim** when authoring or updating any user-facing surface (READMEs, docs, package descriptions). The plan that established them is at `/Users/dipankarsarkar/.claude/plans/cosmic-orbiting-rabin.md`.

**Canonical positioning:**
- Tagline: `The n8n alternative without the bugs — faster, more reliable workflow automation.`
- One-liner: `m9m is a drop-in n8n alternative in Go — 5–10× faster, 70% lower memory, deterministic execution, zero npm dependencies. Run n8n workflows unchanged.`
- Short description (package registries): `Drop-in n8n alternative in Go — faster, more reliable, no Node.js required.`
- Pillars: faster · more reliable · drop-in n8n compatible · agent-native (MCP server built in)
- Reliability story: (1) no JS-runtime memory leaks, (2) deterministic execution, (3) smaller attack surface

**Key Characteristics:**
- **Performance**: 5-10x faster execution than n8n
- **Memory Efficiency**: 70% lower memory usage (~150MB vs 512MB)
- **Container Size**: 75% smaller containers (300MB vs 1.2GB)
- **Startup Time**: Sub-second startup (500ms vs 3s)
- **Reliability**: Single static Go binary, no JS heap leaks, deterministic execution, no npm transitive-dep CVE exposure
- **Architecture**: Cloud-native, microservices-ready

## Technology Stack

- **Language**: Go 1.21+
- **Architecture**: Modular, interface-driven design
- **Queue Systems**: Memory, Redis, RabbitMQ support
- **Monitoring**: Prometheus metrics, OpenTelemetry tracing
- **Testing**: Go standard testing, testify library
- **Build System**: Make-based build automation
- **Deployment**: Docker, Kubernetes-native

## Essential Commands

### Building and Development

Always use the Makefile for consistent builds:

```bash
# Build the application
make build

# Install dependencies
make deps

# Format and validate code
make fmt
make vet
make lint

# Run all tests
make test

# Generate test coverage
make coverage
make coverage-html

# Clean build artifacts
make clean
```

### Testing Commands

Testing should be comprehensive and follow Go conventions:

```bash
# Run all tests
go test ./...

# Run specific package tests
go test ./internal/engine/...

# Run with verbose output
go test -v ./internal/nodes/...

# Run with coverage
go test -cover ./...

# Run integration tests (requires Docker)
./bin/integration-test

# Run benchmarks
./bin/benchmark
```

### Code Quality

Always ensure code quality before committing:

```bash
# Format code (required before commit)
go fmt ./...

# Vet code for common issues
go vet ./...

# Run linter (install golint if needed)
golint ./...

# Run all quality checks
make fmt vet lint test
```

## Project Structure

### Core Architecture

```
m9m/
├── cmd/                    # Application entry points
│   ├── m9m/            # Main application
│   ├── benchmark/         # Performance benchmarking
│   ├── integration-test/  # Integration testing
│   └── template-cli/      # Template management
├── internal/              # Private application code
│   ├── engine/            # Core workflow execution engine
│   ├── nodes/             # Node implementations
│   │   ├── base/          # Base node interfaces and types
│   │   ├── messaging/     # Messaging platform nodes (Slack, Discord, etc.)
│   │   ├── database/      # Database nodes (PostgreSQL, MongoDB, etc.)
│   │   ├── cloud/         # Cloud platform nodes (AWS, GCP, Azure)
│   │   ├── ai/            # AI/LLM nodes (OpenAI, Anthropic)
│   │   ├── transform/     # Data transformation nodes
│   │   └── trigger/       # Trigger nodes (webhooks, timers)
│   ├── monitoring/        # Prometheus metrics and observability
│   ├── credentials/       # Credential management system
│   ├── expressions/       # Expression evaluation engine
│   ├── runtime/           # JavaScript and Python runtime support
│   ├── api/               # REST API implementation
│   ├── queue/             # Queue system implementations
│   └── compatibility/     # n8n compatibility layer
├── docs/                  # Comprehensive documentation
├── examples/              # Example workflows and configurations
├── test-workflows/        # Test workflow definitions
└── Makefile               # Build automation
```

### Key Interfaces and Patterns

#### Node Implementation Pattern

All nodes must implement the `NodeExecutor` interface:

```go
type NodeExecutor interface {
    Execute(inputData []model.DataItem, nodeParams map[string]interface{}) ([]model.DataItem, error)
    Description() NodeDescription
    ValidateParameters(params map[string]interface{}) error
}
```

#### Base Node Usage

Always extend `BaseNode` for common functionality:

```go
type CustomNode struct {
    *base.BaseNode
    // Custom fields
}

func NewCustomNode() *CustomNode {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neul-labs/m9m](https://github.com/neul-labs/m9m) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
