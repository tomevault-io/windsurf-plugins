---
trigger: always_on
description: **Cello** is an ultra-fast, Rust-powered Python async web framework designed to achieve C-level performance on the hot path while maintaining Python's developer experience. It's the successor to frameworks like FastAPI, Robyn, and Litestar, combining their best features with pure Rust implementation for maximum performance.
---

# CLAUDE.md - Cello Framework Project Intelligence

## Project Overview

**Cello** is an ultra-fast, Rust-powered Python async web framework designed to achieve C-level performance on the hot path while maintaining Python's developer experience. It's the successor to frameworks like FastAPI, Robyn, and Litestar, combining their best features with pure Rust implementation for maximum performance.

**Version:** 1.2.4
**License:** MIT
**Python Requirement:** 3.12+
**Author:** Jagadeesh Katla

## Architecture Philosophy

### Core Principle: Rust Owns the Hot Path

```
Request → Rust HTTP Engine → Python Handler → Rust Response
              │                    │
              ├─ SIMD JSON         ├─ Return dict or Response
              ├─ Radix routing     └─ Python business logic only
              └─ Middleware (Rust)
```

**Key Rules:**
- Python = Developer Experience (DX) / DSL
- Rust = Runtime & Execution Engine
- Async-first design
- Zero-copy data flow
- Minimal Python involvement per request

### What Rust Owns (MUST stay in Rust)
- TCP accept loop
- HTTP parsing
- Routing (radix tree)
- All middleware
- JSON serialization (SIMD)
- Response building

### What Python Does (ONLY)
- Route registration
- Handler function pointers
- Business logic
- Returns minimal data structures

## Project Structure

```
/home/vrinda/cello/
├── src/                           # Rust source (23K+ lines, 45 files)
│   ├── lib.rs                     # PyO3 module entry point
│   ├── router.rs                  # Radix-tree routing (matchit)
│   ├── handler.rs                 # Handler registry & caching
│   ├── request/                   # HTTP request handling
│   │   ├── mod.rs                 # Request struct
│   │   ├── body.rs                # Lazy body parsing
│   │   └── multipart.rs           # Multipart form handling
│   ├── response/                  # Response types
│   │   ├── mod.rs                 # Response struct
│   │   ├── streaming.rs           # Streaming responses
│   │   └── xml.rs                 # XML responses
│   ├── middleware/                # Middleware suite (16 files)
│   │   ├── mod.rs                 # Middleware chain & traits
│   │   ├── auth.rs                # JWT, Basic, API Key auth
│   │   ├── rate_limit.rs          # Token bucket, sliding window
│   │   ├── cache.rs               # Smart caching with TTL
│   │   ├── session.rs             # Secure cookie sessions
│   │   ├── security.rs            # CSP, HSTS, security headers
│   │   ├── guards.rs              # RBAC with composable guards
│   │   ├── cors.rs                # CORS handling
│   │   ├── csrf.rs                # CSRF protection
│   │   ├── etag.rs                # ETag caching
│   │   ├── body_limit.rs          # Request size limits
│   │   ├── static_files.rs        # Static file serving
│   │   ├── request_id.rs          # UUID request tracing
│   │   ├── prometheus.rs          # Metrics collection
│   │   ├── circuit_breaker.rs     # Fault tolerance
│   │   ├── exception_handler.rs   # Global error handling
│   │   └── redis.rs               # Redis integration (v0.8.0)
│   ├── routing/                   # Route constraints
│   ├── server/                    # Server modes (cluster, TLS)
│   ├── blueprint.rs               # Flask-like route grouping
│   ├── websocket.rs               # WebSocket support
│   ├── sse.rs                     # Server-Sent Events
│   ├── json.rs                    # SIMD JSON parsing
│   ├── arena.rs                   # Arena allocators
│   ├── context.rs                 # Request context & DI
│   ├── dependency.rs              # Dependency injection
│   ├── error.rs                   # RFC 7807 errors
│   ├── lifecycle.rs               # Startup/shutdown hooks
│   ├── timeout.rs                 # Timeout config
│   ├── dto.rs                     # Data Transfer Objects
│   ├── openapi.rs                 # OpenAPI generation
│   ├── background.rs              # Background tasks
│   └── template.rs                # Jinja2 templates
│
├── python/cello/                  # Python API wrapper
│   ├── __init__.py                # Public Python API
│   ├── database.py                # Database & Redis wrappers (v0.8.0)
│   ├── guards.py                  # RBAC guard classes
│   └── validation.py              # DTO validation
│
├── tests/                         # Test suite
│   ├── test_cello.py              # Main integration tests
│   └── verify_*.py                # Feature verification tests
│
├── examples/                      # 20 example applications
│   ├── hello.py                   # Basic hello world
│   ├── simple_api.py              # REST API with OpenAPI
│   ├── comprehensive_demo.py      # All v0.7.0 features
│   ├── database_demo.py           # Database & Redis (v0.8.0)
│   ├── guards.py                  # RBAC examples
│   └── ...
│
├── docs/                          # Documentation
│   ├── README.md                  # Doc index
│   ├── getting-started.md         # Installation & basics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jagadeesh32/cello](https://github.com/jagadeesh32/cello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
