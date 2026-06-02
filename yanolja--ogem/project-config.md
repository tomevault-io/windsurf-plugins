---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ogem is a production-ready AI proxy server that provides unified access to multiple AI language models through an OpenAI-compatible API. It acts as an intelligent gateway between clients and various AI providers (OpenAI, Claude, Gemini, etc.) with enterprise features including routing, caching, multi-tenancy, rate limiting, and security.

## Development Commands

### Building and Running
```bash
# Build the binary
go build -o ogem cmd/main.go

# Run locally (uses config.yaml by default)
go run cmd/main.go

# Run with custom config
go run cmd/main.go -config path/to/config.yaml

# Run with Docker
docker build -t ogem:latest .
docker run -p 8080:8080 -e OPEN_GEMINI_API_KEY="your-key" ogem:latest
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests with race detection
go test -race ./...

# Run tests for specific package
go test ./cache
go test ./routing
go test ./provider

# Run comprehensive test coverage analysis
./scripts/test-coverage.sh

# Run specific test categories
./scripts/test-coverage.sh unit          # Unit tests only
./scripts/test-coverage.sh integration   # Integration tests only
./scripts/test-coverage.sh e2e          # End-to-end tests only

# Run single test
go test ./cache -run TestCacheHit
```

### Linting
```bash
# Requires golangci-lint to be installed
golangci-lint run

# Auto-fix issues where possible
golangci-lint run --fix
```

## Architecture Overview

### Request Flow

1. **Entry Point** (`server/server.go`): HTTP server receives OpenAI-compatible requests
2. **Authentication** (`auth/`): Validates API keys (master keys, virtual keys, JWT, OAuth2)
3. **Multi-Tenancy** (`tenancy/`): Identifies tenant and applies tenant-specific quotas/limits
4. **Security** (`security/`): Rate limiting, PII masking, audit logging
5. **Cache Layer** (`cache/`): Checks for cached responses (exact, semantic, token-based)
6. **Router** (`routing/`): Selects optimal AI provider endpoint based on latency, cost, load
7. **Provider** (`provider/`): Transforms request to provider-specific format and executes
8. **Response**: Flows back through cache, monitoring, cost tracking

### Core Packages

**provider/**: Abstract interface `AiEndpoint` with implementations for each AI provider
- `openai/`: Direct OpenAI API integration
- `claude/`: Anthropic Claude API integration
- `vertex/`: Google Vertex AI (GCP-hosted Gemini models)
- `studio/`: Google AI Studio (Gemini API)
- `vclaude/`: Claude models via Vertex AI
- Each provider implements the same interface but handles provider-specific protocols

**routing/**: Intelligent request routing with multiple strategies
- `StrategyLatency`: Routes to fastest endpoint (default)
- `StrategyCost`: Routes to cheapest endpoint
- `StrategyRoundRobin`: Even distribution
- `StrategyLeastConnections`: Routes to endpoint with fewest active requests
- `StrategyPerformanceBased`: Weighted combination of cost, latency, success rate, load
- `StrategyAdaptive`: Dynamically switches strategies based on conditions
- Includes circuit breaker pattern to handle failing endpoints

**cache/**: Multi-strategy caching system
- `StrategyExact`: Caches exact request matches
- `StrategySemantic`: Semantic similarity matching
- `StrategyToken`: Token-level similarity with fuzzy matching
- `StrategyHybrid`: Combines multiple strategies
- `StrategyAdaptive`: Dynamically adjusts based on hit rates
- Supports memory and Redis backends

**auth/**: Authentication system with multiple methods
- Master API key authentication (OPEN_GEMINI_API_KEY)
- Virtual keys with granular permissions and spend limits
- JWT authentication
- OAuth2 integration
- Priority: virtual keys checked first, then master key

**tenancy/**: Multi-tenant isolation and resource management
- Hierarchical tenant structure (enterprise → team → user)
- Per-tenant quotas, rate limits, cost budgets
- Usage tracking and billing integration
- Tenant-specific configuration overrides

**security/**: Security features
- Rate limiting (per tenant, per key)
- PII masking in logs and responses
- Audit logging of all requests
- Security manager coordinates all security policies

**monitoring/**: Observability and metrics
- Prometheus metrics export
- OpenTelemetry integration
- Datadog integration
- Custom metrics backend support

**state/**: Distributed state management
- Memory-based state (single instance)
- Valkey/Redis state (distributed deployment)
- Used for rate limiting, quotas, request tracking

**cost/**: Cost calculation and tracking
- Per-model pricing (input tokens, output tokens, reasoning tokens)
- Real-time cost tracking per request
- Budget enforcement
- Pricing updated for 2025 models

### Configuration System

Configuration is loaded from YAML files with environment variable overrides:
- `CONFIG_SOURCE`: Path or URL to config file (default: config.yaml)
- `CONFIG_TOKEN`: Bearer token for authenticated remote configs
- Provider credentials via environment variables (OPENAI_API_KEY, CLAUDE_API_KEY, etc.)
- `VALKEY_ENDPOINT`: Redis-compatible endpoint for distributed state

Configuration supports:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yanolja/ogem](https://github.com/yanolja/ogem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
