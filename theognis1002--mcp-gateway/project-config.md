---
trigger: always_on
description: The **MCP Gateway** is a production-ready API gateway for Model Context Protocol (MCP) servers, providing enterprise-grade infrastructure with authentication, logging, rate limiting, server discovery, and multi-protocol transport support.
---

# MCP Gateway - Claude Development Guide

## Project Overview

The **MCP Gateway** is a production-ready API gateway for Model Context Protocol (MCP) servers, providing enterprise-grade infrastructure with authentication, logging, rate limiting, server discovery, and multi-protocol transport support.

### Core Purpose
- **Enterprise Infrastructure**: JWT authentication, RBAC, and flexible policies
- **Multi-Protocol Support**: JSON-RPC, WebSocket, SSE, HTTP, and STDIO transports
- **Service Virtualization**: Wrap REST/GraphQL/gRPC services as virtual MCP servers
- **Production Ready**: Comprehensive logging, IP rate limiting with Redis/memory backends, and health monitoring
- **Namespace Management**: Group MCP servers into logical namespaces within an organization
- **Developer Tools**: MCP Inspector for real-time debugging and testing

### Key Features
- 🔐 **Authentication & Authorization** - JWT-based auth with API keys, RBAC, and OAuth2
- 📊 **Comprehensive Logging** - Request/response logging, audit trails, performance metrics
- ⚡ **Rate Limiting** - IP-based rate limiting with sliding window algorithms
- 🛡️ **Smart Rate Limiting** - Redis-backed with memory fallback and proxy detection
- 🔍 **MCP Server Discovery** - Dynamic registration and health checking
- 🌐 **Service Virtualization** - Wrap non-MCP services as virtual MCP servers
- 🔌 **Multi-Protocol Support** - JSON-RPC, WebSocket, SSE, HTTP, and STDIO transports
- 🚀 **High Performance** - Built with Go and Gin for maximum throughput and low latency
- 🏢 **Namespace Management** - Group and organize MCP servers into logical namespaces
- 🔍 **MCP Inspector** - Real-time debugging and testing interface
- 🤝 **Agent-to-Agent (A2A)** - Agent-to-agent authentication and communication
- 🔧 **Plugin System** - Extensible middleware with content filters and AI integrations
- 📡 **Endpoint Management** - Dynamic REST endpoint creation with OpenAPI generation
- 🛠️ **Content Filtering** - PII detection, regex filtering, and resource protection

## Architecture

### Technology Stack
- **Backend**: Go 1.25 with Gin framework
- **Database**: PostgreSQL with comprehensive migration system
- **Cache**: Redis (optional, falls back to in-memory when disabled)
- **Frontend**: Next.js 14 TypeScript dashboard
- **Testing**: Extensive test suites for all transport layers

### Architectural Decisions
- **Rate Limiting**: Uses industry-standard `ulule/limiter` library instead of custom implementations
- **Redis Integration**: Sliding window rate limiting for distributed deployments with memory fallback
- **Middleware Pattern**: Composable middleware chain for cross-cutting concerns
- **Clean Architecture**: Separation of concerns with internal packages for different domains

### Project Structure
```
mcp-gateway/
├── apps/
│   ├── backend/              # Go API backend
│   │   ├── cmd/
│   │   │   ├── api/          # API server entrypoint (main.go)
│   │   │   ├── migrate/      # Database migration tool
│   │   │   └── worker/       # Background worker for health checks
│   │   ├── internal/         # Core business logic modules
│   │   │   ├── auth/         # Authentication & Authorization
│   │   │   │   ├── jwt.go    # JWT token management
│   │   │   │   ├── middleware.go # Auth middleware
│   │   │   │   ├── policies.go # Policy engine
│   │   │   │   ├── rbac.go   # Role-based access control
│   │   │   │   ├── cache.go  # JWT blacklist cache
│   │   │   │   ├── cleanup.go # Token cleanup service
│   │   │   │   └── service.go # Auth service
│   │   │   ├── a2a/          # Agent-to-Agent Auth
│   │   │   │   ├── service.go # A2A service
│   │   │   │   ├── client.go # A2A client
│   │   │   │   └── adapter.go # A2A adapters
│   │   │   ├── config/       # Configuration management
│   │   │   │   ├── config.go # Config structs and loading
│   │   │   │   ├── policy.go # Policy configuration
│   │   │   │   └── validation.go # Config validation
│   │   │   ├── database/     # Database layer
│   │   │   │   ├── database.go # Database connection
│   │   │   │   ├── models/   # Database models
│   │   │   │   │   ├── base.go # Base model
│   │   │   │   │   ├── user.go # User model
│   │   │   │   │   ├── organization.go # Organization model
│   │   │   │   │   ├── namespace.go # Namespace model
│   │   │   │   │   ├── server.go # MCP server model
│   │   │   │   │   ├── session.go # Session model
│   │   │   │   │   ├── virtual_server.go # Virtual server model
│   │   │   │   │   ├── a2a.go   # Agent-to-Agent models
│   │   │   │   │   ├── config.go # Configuration models
│   │   │   │   │   ├── content_filter.go # Content filter models
│   │   │   │   │   ├── logging.go # Logging models
│   │   │   │   │   └── ratelimit.go # Rate limiting models
│   │   │   │   └── repositories/ # Database repositories
│   │   │   │       └── namespace_repo.go # Namespace repository
│   │   │   ├── discovery/    # MCP Server Discovery
│   │   │   │   ├── health.go # Health checking
│   │   │   │   ├── registry.go # Server registry
│   │   │   │   ├── mcp_discovery.go # MCP discovery service

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theognis1002/mcp-gateway](https://github.com/theognis1002/mcp-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
