---
trigger: always_on
description: CCProxy is a high-performance Go-based API translation proxy that enables Claude Code to work with multiple AI providers. It acts as a middleware layer that translates Anthropic's API format to various provider-specific formats, enabling seamless integration without modifying Claude Code. This is a complete rewrite from TypeScript to Go with enhanced security, performance, and reliability features.
---

# CCProxy - AI Request Proxy for Claude Code (Go Implementation)

## Project Overview

CCProxy is a high-performance Go-based API translation proxy that enables Claude Code to work with multiple AI providers. It acts as a middleware layer that translates Anthropic's API format to various provider-specific formats, enabling seamless integration without modifying Claude Code. This is a complete rewrite from TypeScript to Go with enhanced security, performance, and reliability features.

**Important**: CCProxy is a translation proxy - it does not add new capabilities beyond API format translation. It requires models with function calling support for Claude Code compatibility.

## Architecture

### Core Components

- **CLI Interface** (`cmd/ccproxy/`) - Cobra-based command-line interface
- **Server** (`internal/server/`) - Gin-based HTTP server with middleware stack
- **Pipeline** (`internal/pipeline/`) - Request processing pipeline with transformations
- **Providers** (`internal/providers/`) - Multi-provider AI service integration
- **Transformers** (`internal/transformer/`) - Request/response transformation engine
- **Router** (`internal/router/`) - Intelligent model routing based on token count and parameters
- **Performance** (`internal/performance/`) - Monitoring, rate limiting, and resource management
- **Security** (`internal/security/`) - Authentication, authorization, and audit logging

### Key Features

- **API Translation**: Converts Anthropic API format to provider-specific formats
- **Intelligent Routing**: Automatic model selection based on:
  - Token count (>60K → longContext route)
  - Model type (haiku models → background route)
  - Boolean thinking parameter (thinking: true → think route)
  - Explicit provider selection (format: "provider,model")
- **Provider Support**: Full transformers for Anthropic, OpenAI, Gemini, DeepSeek, OpenRouter
- **Streaming Support**: Server-Sent Events (SSE) for real-time responses
- **Process Management**: Background service with PID file locking and graceful shutdown
- **Claude Code Integration**: Auto-start, environment variable management, reference counting

## Security Features

### Authentication & Authorization
- API key validation (Bearer token and x-api-key header)
- Localhost-only enforcement when no API key configured
- IP-based access controls with whitelist/blacklist
- Health endpoints with graduated access (basic status public, details require auth)

### Security Hardening
- Request size limits (10MB default) to prevent DoS attacks
- Configurable timeouts (30 seconds default) instead of 60-minute hardcoded values
- Provider error response sanitization (removes API keys, tokens, emails)
- CORS headers sanitized (x-api-key removed from public headers)
- Resource limit enforcement with circuit breaker patterns

### Process Security
- Exclusive PID file locking to prevent multiple instances
- No unsafe fallbacks - fails fast if locking unavailable
- Atomic operations for metrics tracking to prevent race conditions

## Configuration

### Default Configuration
```json
{
  "host": "127.0.0.1",
  "port": 3456,
  "performance": {
    "request_timeout": "30s",
    "max_request_body_size": 10485760,
    "metrics_enabled": true,
    "rate_limit_enabled": false,
    "circuit_breaker_enabled": true
  }
}
```

### Environment Variables
- `CCPROXY_PORT` - Override default port
- `CCPROXY_HOST` - Override default host  
- `CCPROXY_API_KEY` - Set API key for authentication
- `CCPROXY_CONFIG` - Path to configuration file
- `ANTHROPIC_API_KEY` - Anthropic Claude API key (auto-detected)
- `OPENAI_API_KEY` - OpenAI API key (auto-detected)
- `GEMINI_API_KEY` - Google Gemini API key (auto-detected)
- `DEEPSEEK_API_KEY` - DeepSeek API key (auto-detected)
- `LOG` - Enable file logging to ~/.ccproxy/ccproxy.log

## Commands

### Primary Commands
- `ccproxy start` - Start the service in background
- `ccproxy stop` - Stop the running service
- `ccproxy status` - Show service status with emoji indicators
- `ccproxy code` - Auto-start service and configure Claude Code environment
- `ccproxy claude` - Manage ~/.claude.json configuration
- `ccproxy env` - Show environment variable documentation
- `ccproxy version` - Show version information

### Command Options
- `--config` - Specify custom configuration file
- `--foreground` - Run service in foreground (for start command)

## Build & Development

### Building
```bash
make build              # Build for current platform
make build-all          # Build for all platforms
make docker-build       # Build Docker image
make test               # Run tests
make test-race          # Run tests with race detection
```

### Testing
- **Unit Tests**: Comprehensive coverage for all components
- **Integration Tests**: End-to-end request processing
- **Race Detection**: All tests pass with `-race` flag
- **Benchmark Tests**: Performance validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orchestre-dev/ccproxy](https://github.com/orchestre-dev/ccproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
