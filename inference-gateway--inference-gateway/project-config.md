---
trigger: always_on
description: > **Last Updated**: May 8, 2026
---

# AGENTS.md - Inference Gateway

> **Last Updated**: May 8, 2026
> **Repository**: github.com/inference-gateway/inference-gateway
> **Current Version**: v0.24.1

This document provides comprehensive guidance for AI agents working with the Inference Gateway project.
It covers project architecture, development workflow, testing patterns, code generation, and conventions.

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture and Structure](#architecture-and-structure)
- [Development Environment Setup](#development-environment-setup)
- [Key Commands](#key-commands)
- [Testing Instructions](#testing-instructions)
- [Provider System](#provider-system)
- [Code Generation Workflow](#code-generation-workflow)
- [Project Conventions](#project-conventions)
- [Important Files](#important-files)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)

## Project Overview

**Inference Gateway** is a unified API proxy server for multiple LLM providers with Model Context Protocol (MCP) integration,
OpenTelemetry metrics, and enterprise-ready features. It provides an OpenAI-compatible API surface that routes requests to
various LLM providers.

### Key Technologies

- **Language**: Go 1.26.2
- **HTTP Framework**: Gin (`github.com/gin-gonic/gin`)
- **Code Generation**: OpenAPI-driven custom generator + oapi-codegen
- **Configuration**: Environment-based (`sethvargo/go-envconfig`)
- **Logging**: Structured logging via zap (`go.uber.org/zap`)
- **Telemetry**: OpenTelemetry with Prometheus exporter
- **MCP**: Model Context Protocol (`metoro-io/mcp-golang`)
- **Authentication**: OIDC with `coreos/go-oidc/v3`
- **Mocking**: `go.uber.org/mock/mockgen`
- **Testing**: `stretchr/testify` + gomock
- **Task Runner**: Task (Taskfile.yml)
- **Release**: semantic-release + goreleaser
- **Containerization**: Docker, Docker Compose, Kubernetes (Helm)
- **Monitoring**: Prometheus, Grafana

## Architecture and Structure

### Directory Layout

```text
api/                        # HTTP API layer
├── middlewares/            # Auth, Logger, MCP, Telemetry middleware
└── routes.go               # Route handlers (chat, models, proxy, tools, health)
cmd/                        # Entry points
├── gateway/main.go         # Main gateway server
└── generate/main.go        # Code generation tool CLI
config/                     # Configuration (generated from OpenAPI)
├── config.go               # Config struct, Load(), provider defaults
├── config_test.go
└── meta.go                 # APPLICATION_NAME, VERSION constants
providers/                  # LLM provider implementations
├── client/client.go        # HTTP client config & interface
├── constants/              # Provider IDs, URLs, auth types (GENERATED)
├── core/                   # IProvider interface & ProviderImpl base
├── registry/               # Provider registry & config (GENERATED)
├── routing/                # Model string → provider prefix mapping
├── transformers/           # Provider-specific response transformers
└── types/                  # Shared OpenAI-compatible types (GENERATED)
mcp/                        # Model Context Protocol
├── agent.go                # MCP agent - tool execution & iteration loop
├── client.go               # MCP HTTP client with SSE transport
├── generated_types.go      # MCP schema types (GENERATED)
├── mcp-schema.json/yaml    # MCP schema definitions
internal/                   # Internal generators
├── codegen/codegen.go      # Custom code generator
├── dockergen/dockergen.go  # Docker env example generator
├── kubegen/kubegen.go      # K8s manifest generator
├── mdgen/mdgen.go          # Markdown doc generator
├── openapi/openapi.go      # OpenAPI spec parser
└── proxy/proxy.go          # Dev-mode proxy modifiers
logger/                     # Logger interface + zap implementation
otel/                       # OpenTelemetry metrics implementation
tests/                      # All tests + mocks
├── api_routes_test.go      # Route handler tests
├── providers_test.go       # Provider tests
├── mcp_agent_test.go       # MCP agent tests
├── mcp_enhanced_test.go    # Enhanced MCP tests
├── multimodal_test.go      # Vision/multimodal tests
├── logger_test.go
├── middlewares/mcp_test.go  # MCP middleware tests
└── mocks/                  # Generated mocks
examples/                   # Deployment examples
├── docker-compose/         # basic, hybrid, mcp, auth, monitoring, tools
└── kubernetes/             # basic, hybrid, mcp, agent, auth, monitoring, tls
charts/inference-gateway/   # Helm chart
scripts/                    # Pre-commit hook
hack/                       # Dev cluster management
```

### Core Components

#### 1. Gateway Server (`cmd/gateway/main.go`)

- Initializes Config (env-based), Logger (zap), Telemetry (OTel Prometheus)
- Sets up provider registry and HTTP client
- Configures middleware pipeline: Logger → Telemetry → OIDC Auth → MCP
- Registers routes on Gin engine
- Supports TLS and graceful shutdown via OS signals

#### 2. API Layer (`api/routes.go`)

- `RouterImpl` implements the `Router` interface
- Routes: `GET /health`, `GET /v1/models`, `POST /v1/chat/completions`, `GET /v1/mcp/tools`, `/proxy/:provider/*path`
- Handles both streaming (SSE) and non-streaming (JSON) responses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inference-gateway/inference-gateway](https://github.com/inference-gateway/inference-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
