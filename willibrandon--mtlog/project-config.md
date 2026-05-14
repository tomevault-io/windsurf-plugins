---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**mtlog** (Message Template Logging) is a high-performance, Serilog-inspired structured logging library for Go. The library brings message templates and pipeline architecture to the Go ecosystem, with native integration for Seq, Elasticsearch, and Splunk.

## Key Features

### 1. Message Templates
- Templates like `"User {UserId} logged in"` are preserved throughout the pipeline
- Properties are extracted from templates and matched positionally to arguments
- Templates serve as both human-readable messages and event types for grouping/analysis
- Support for format specifiers like `{Count:000}` and `{Price:F2}`
- OTEL-compatible dotted property names like `{http.method}`, `{service.name}`, `{db.system}`

### 2. Output Templates (v0.6.0+)
- Output templates use `${...}` syntax for built-in elements to avoid conflicts
- Built-in elements: `${Timestamp}`, `${Level}`, `${Message}`, `${Exception}`, `${NewLine}`, `${Properties}`
- User properties continue to use `{...}` syntax: `{UserId}`, `{RequestId}`, etc.
- Example: `"[${Timestamp}] ${Level} ${Message} {UserId}"` - clearly distinguishes built-ins from user properties

### 3. Pipeline Architecture
The logging pipeline follows this flow:
```
Message Template Parser → Enrichment → Filtering → Capturing → Sinks (Output)
```

### 4. Ecosystem Compatibility
- **slog**: Full compatibility with Go's standard `log/slog` package via `slog.Handler` adapter
- **logr**: Integration with Kubernetes ecosystem via `logr.LogSink` adapter
- **Short Methods**: Convenience methods like `V()`, `D()`, `I()`, `W()`, `E()`, `F()`

### 5. Core Interfaces
- `Logger` - Main logging interface with methods like `Information()`, `Error()`, etc.
- `LogEventEnricher` - Adds contextual properties to log events
- `LogEventFilter` - Determines which events proceed through pipeline
- `Capturer` - Converts complex types to log-appropriate representations
- `LogEventSink` - Outputs events to destinations (Console, File, Seq, etc.)
- `LoggingLevelSwitch` - Dynamic level control for runtime configuration

### 6. SelfLog Diagnostics
- Internal diagnostic facility for debugging silent failures
- Zero-cost when disabled (0.37ns/op with guard check)
- Outputs to any `io.Writer` or custom function
- Environment variable support: `MTLOG_SELFLOG=stderr/stdout/file`
- Reports sink failures, template errors, panic recovery, and configuration issues

### 7. Template Cache (v0.8.1+)
- **LRU cache** for parsed message templates to avoid repeated allocations
- **Bounded size** (default: 10,000) to prevent memory exhaustion from dynamic templates
- **Sharded design** with up to 64 shards for concurrent access
- **O(1) operations** with proper LRU eviction
- **Optional TTL** support for time-based expiration
- **Thread-safe** with atomic statistics tracking
- **Security fix** for issue #39 - prevents DoS via unbounded template generation

## Development Commands

```bash
# Run all tests
go test ./...

# Run with coverage
go test -cover ./...

# Run only integration tests (requires Docker)
go test -tags=integration ./...

# Run benchmarks
go test -bench=. -benchmem ./...

# Run with race detector
go test -race ./...

# Run specific test
go test -run TestSeqIntegration ./...

# Run fuzz tests
go test -fuzz=FuzzParseMessageTemplate -fuzztime=30s ./parser

# Format code
go fmt ./...

# Run linter
golangci-lint run

# Run benchmarks with specific focus
go test -bench=BenchmarkSimpleString -benchmem -benchtime=10s .

# Run mtlog-analyzer tests
cd cmd/mtlog-analyzer && go test -v ./...

# Run mtlog-analyzer on the project
go vet -vettool=$(which mtlog-analyzer) ./...
```

## mtlog-analyzer

The project includes a static analysis tool that catches common mtlog mistakes at compile time:

### Installation
```bash
go install github.com/willibrandon/mtlog/cmd/mtlog-analyzer@latest
```

### Features
- Template/argument mismatch detection
- Format specifier validation
- Property naming conventions (PascalCase suggestions)
- Duplicate property detection
- Capturing hints for complex types
- Error logging pattern validation
- Context key constant suggestions

### Usage
```bash
# Run with go vet
go vet -vettool=$(which mtlog-analyzer) ./...

# Run standalone
mtlog-analyzer ./...

# With configuration flags
mtlog-analyzer -strict -common-keys=tenant_id,org_id ./...
```

### Configuration Flags
- `-strict` - Enable strict format specifier validation
- `-common-keys` - Additional context keys to suggest as constants
- `-disable` - Disable specific checks (template, naming, etc.)
- `-ignore-dynamic-templates` - Suppress warnings for non-literal templates
- `-strict-logger-types` - Only analyze exact mtlog types
- `-downgrade-errors` - Downgrade errors to warnings for CI migration

### IDE Integration

#### VS Code Extension
The mtlog-analyzer is integrated into VS Code through the official extension:
- Real-time validation with inline diagnostics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willibrandon/mtlog](https://github.com/willibrandon/mtlog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
