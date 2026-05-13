---
trigger: always_on
description: Olla is a high-performance proxy and load balancer for LLM infrastructure, written in Go. It intelligently routes requests across local and remote inference nodes (Ollama, LM Studio, LiteLLM, vLLM, SGLang, Llamacpp, Lemonade, Anthropic, and OpenAI-compatible endpoints).
---

# CLAUDE.md

## Overview
Olla is a high-performance proxy and load balancer for LLM infrastructure, written in Go. It intelligently routes requests across local and remote inference nodes (Ollama, LM Studio, LiteLLM, vLLM, SGLang, Llamacpp, Lemonade, Anthropic, and OpenAI-compatible endpoints).

The project provides two proxy engines: Sherpa (simple, maintainable) and Olla (high-performance with advanced features).

Full documentation available at: https://thushan.github.io/olla/

## Commands
```bash
make ready           # Run before commit (test-short + test-race + fmt + vet + lint + align)
make ready-tools     # Check code with tools only (fmt + vet + lint + align)
make test            # Run all tests
make test-race       # Run tests with race detection
make test-stress     # Run comprehensive stress tests
make bench           # Run all benchmarks
make bench-balancer  # Run balancer benchmarks
make build           # Build optimised binary with version info
make build-local     # Build binary to ./build/ (fast, for testing)
make run             # Run with version info
make run-debug       # Run with debug logging
make docker-build    # Build Docker image with goreleaser (requires goreleaser)
make docker-build-local # Build Docker image locally for amd64 (no goreleaser required)
make docker-build-local-arm64 # Build Docker image locally for ARM64 (no goreleaser required)
make docker-run      # Run Docker image with local config (amd64)
make ci              # Run full CI pipeline locally
make help            # Show all available targets
```

## Project Structure
```
olla/
├── main.go                    # Entry point, initialises services
├── config.yaml               # Default configuration
├── config/
│   ├── profiles/            # Provider-specific profiles
│   │   ├── ollama.yaml     # Ollama configuration
│   │   ├── llamacpp.yaml   # llama.cpp configuration
│   │   ├── lmstudio.yaml   # LM Studio configuration
│   │   ├── lemonade.yaml   # Lemonade SDK configuration
│   │   ├── litellm.yaml    # LiteLLM gateway configuration
│   │   ├── vllm.yaml       # vLLM configuration
│   │   ├── sglang.yaml     # SGLang configuration
│   │   ├── openai-compatible.yaml  # OpenAI-compatible generic profile
│   │   └── openai.yaml     # OpenAI profile
│   ├── models.yaml         # Model configurations
│   └── config.local.yaml   # Local configuration overrides (user, not committed to git)
├── internal/
│   ├── core/               # Domain layer (business logic)
│   │   ├── domain/         # Core entities
│   │   ├── ports/          # Interface definitions
│   │   └── constants/      # Application constants
│   ├── adapter/            # Infrastructure layer
│   │   ├── balancer/       # Load balancing strategies
│   │   ├── converter/      # Model format converters
│   │   ├── discovery/      # Service discovery
│   │   ├── factory/        # Factory patterns
│   │   ├── filter/         # Request/response filtering
│   │   ├── health/         # Health checking & circuit breakers
│   │   ├── inspector/      # Request inspection
│   │   ├── metrics/        # Metrics collection
│   │   ├── proxy/          # Proxy implementations
│   │   │   ├── sherpa/     # Simple, maintainable proxy
│   │   │   ├── olla/       # High-performance proxy
│   │   │   └── core/       # Shared proxy components
│   │   ├── registry/       # Model & profile registries
│   │   ├── security/       # Security features (rate/size limits)
│   │   ├── stats/          # Statistics collection
│   │   ├── translator/     # API translation layer (OpenAI ↔ Provider)
│   │   └── unifier/        # Model unification
│   ├── app/                # Application layer
│   │   ├── handlers/       # HTTP handlers
│   │   │   ├── server.go              # HTTP server setup
│   │   │   ├── server_routes.go       # Route registration
│   │   │   ├── handler_proxy.go       # Main proxy handler
│   │   │   ├── handler_provider_*.go  # Provider-specific handlers
│   │   │   ├── handler_translation.go # Translation handler
│   │   │   ├── handler_status*.go     # Status endpoints
│   │   │   ├── handler_health.go      # Health endpoints
│   │   │   └── handler_version.go     # Version information
│   │   ├── middleware/     # HTTP middleware
│   │   └── services/       # Application services
│   ├── config/             # Configuration management
│   ├── env/                # Environment handling
│   ├── integration/        # Integration tests
│   ├── logger/             # Logging framework
│   ├── router/             # Routing logic
│   ├── util/               # Utilities
│   └── version/            # Version management
├── pkg/                    # Reusable packages
│   ├── container/         # Dependency injection
│   ├── eventbus/          # Event bus (pub/sub)
│   ├── format/            # Formatting utilities
│   ├── nerdstats/         # Process statistics
│   ├── pool/              # Object pooling
│   └── profiler/          # Profiling support
└── test/
    └── scripts/           # Test scripts
        ├── auth/          # Authentication tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thushan/olla](https://github.com/thushan/olla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
