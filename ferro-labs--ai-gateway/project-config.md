---
trigger: always_on
description: **Ferro Labs AI Gateway** is a high-performance, open-source AI gateway written in Go. It acts as a unified routing layer between applications and 100+ LLM providers (OpenAI, Anthropic, Gemini, Mistral, etc.), offering smart routing, plugin middleware, and API key management — all with an OpenAI-compatible API and transparent pass-through proxy.
---

# AGENTS.md

## Project Overview

**Ferro Labs AI Gateway** is a high-performance, open-source AI gateway written in Go. It acts as a unified routing layer between applications and 100+ LLM providers (OpenAI, Anthropic, Gemini, Mistral, etc.), offering smart routing, plugin middleware, and API key management — all with an OpenAI-compatible API and transparent pass-through proxy.

- **Module**: `github.com/ferro-labs/ai-gateway`
- **Go version**: 1.24+
- **License**: Apache 2.0

### Current Development Snapshot

- **29 provider subpackages** — each provider lives in `providers/<id>/<id>.go` with its own test file. No root-level constructor shims remain.
- **Unified factory** — `providers/factory.go` holds types/constants; `providers/providers_list.go` holds all built-in `ProviderEntry` records. Auto-registration via `AllProviders()` means `main.go` never needs editing for new providers.
- **`providers/core/` split** — interfaces in `contracts.go`; shared types split into `chat.go`, `stream.go`, `embedding.go`, `image.go`, `model.go`, `constants.go`, `errors.go`.
- **Single source of truth for name constants** — `providers/names.go` re-exports `NameXxx` from each subpackage's `const Name`.
- **`internal/discovery/`** — shared OpenAI-compatible model discovery helper used by fireworks, hugging_face, perplexity, xai.
- **Provider coverage** — OpenAI, Anthropic, Gemini, Groq, Bedrock, Vertex AI, Hugging Face, Cerebras, Cloudflare, Databricks, DeepInfra, Moonshot, Novita, NVIDIA NIM, OpenRouter, Qwen, SambaNova, and more.
- **Built-in OSS plugins** — word filter, max token, response cache, request logger, rate limit, budget.
- **Admin API** — dashboard, key management, usage stats, request logs, config history/rollback (`internal/admin/handlers.go`).
- **Metrics** — Prometheus metrics exposed at `/metrics` (`internal/metrics/`).
- **Circuit breaker** — per-provider circuit breaker in `internal/circuitbreaker/`.

---

## Build, Test, and Run Commands

```bash
# Build
make build          # builds ./bin/ferrogw
make all            # fmt + lint + test + coverage + build

# Run
make run            # requires at least one provider key, e.g. OPENAI_API_KEY=sk-...

# Test
make test           # unit tests
make test-coverage  # with coverage report
make test-integration  # requires provider API keys

# Code quality
make fmt            # gofmt
make lint           # golangci-lint
make precommit      # fmt + test

# Docker
docker-compose up   # local dev environment
```

---

## Project Structure

```sh
ai-gateway/
├── cmd/
│   └── ferrogw/          # HTTP server + CLI entry point (Cobra subcommands)
│       ├── main.go       # Server setup, provider registration, router, Cobra root
│       ├── cors.go       # CORS middleware
│       ├── completions.go # Legacy /v1/completions handler
│       └── proxy.go      # Pass-through proxy for /v1/*
├── internal/
│   ├── admin/            # API key management + auth middleware
│   ├── cache/            # Cache interface + in-memory implementation
│   ├── cli/              # Shared CLI command implementations (doctor, status, admin, etc.)
│   ├── plugins/          # Built-in plugin implementations
│   │   ├── cache/        # Request/response caching
│   │   ├── logger/       # Request/response logging
│   │   ├── maxtoken/     # Token/message limit guardrail
│   │   └── wordfilter/   # Blocked word guardrail
│   ├── strategies/       # Routing strategy implementations
│   └── version/
├── plugin/               # Public plugin framework (interfaces + manager + registry)
├── providers/
│   ├── core/             # Shared interfaces (contracts.go) and types (chat, stream, embedding, image, model)
│   ├── <id>/             # One subpackage per provider
│   ├── factory.go        # ProviderConfig, ProviderEntry, CfgKey* & Capability* consts, lookup funcs
│   ├── providers_list.go # allProviders slice — all built-in ProviderEntry registrations
│   ├── names.go          # NameXxx constants (re-exported from each subpackage)
│   ├── registry.go       # Registry type for runtime lookup by name
│   └── facade_aliases.go # Type aliases re-exporting core.* for backwards compatibility
├── internal/
│   ├── admin/            # API key management, dashboard, logs, config history
│   ├── cache/            # Cache interface + in-memory implementation
│   ├── circuitbreaker/   # Per-provider circuit breaker
│   ├── discovery/        # Shared OpenAI-compatible model discovery helper
│   ├── latency/          # Latency tracking for least-latency strategy
│   ├── metrics/          # Prometheus metrics
│   ├── plugins/          # Built-in plugin implementations
│   │   ├── cache/        # Request/response caching
│   │   ├── logger/       # Request/response logging
│   │   ├── maxtoken/     # Token/message limit guardrail
│   │   ├── ratelimit/    # Rate limiting
│   │   └── wordfilter/   # Blocked word guardrail
│   ├── ratelimit/        # Rate limit internals

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ferro-labs/ai-gateway](https://github.com/ferro-labs/ai-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
