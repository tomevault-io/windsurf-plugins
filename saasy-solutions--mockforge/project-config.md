---
trigger: always_on
description: MockForge is a high-performance mock API server written in Rust. It's a workspace of 42 crates supporting 10+ protocols (HTTP, gRPC, WebSocket, GraphQL, Kafka, MQTT, AMQP, SMTP, FTP, TCP), with an admin UI (React 19 + TypeScript), plugin system, benchmarking (k6), and a multi-tenant registry server.
---

# MockForge — Claude Code Guide

## Project Overview

MockForge is a high-performance mock API server written in Rust. It's a workspace of 42 crates supporting 10+ protocols (HTTP, gRPC, WebSocket, GraphQL, Kafka, MQTT, AMQP, SMTP, FTP, TCP), with an admin UI (React 19 + TypeScript), plugin system, benchmarking (k6), and a multi-tenant registry server.

- **Language**: Rust 2021 edition, `unsafe_code = "deny"` workspace-wide
- **Version**: 0.3.31+ (workspace.package.version in root Cargo.toml)
- **License**: MIT OR Apache-2.0
- **Repository**: https://github.com/SaaSy-Solutions/mockforge

## Quick Start

```bash
# Build
cargo build --workspace

# Run with example spec (HTTP :3000, WS :3001, gRPC :50051, Admin UI :9080)
make run-example

# Run tests
cargo test --workspace

# UI dev server
cd crates/mockforge-ui/ui && pnpm install && pnpm dev
```

## Running MockForge

The main binary is `mockforge-cli`. Key environment variables and flags:

```bash
# Minimal: serve an OpenAPI spec on HTTP
cargo run -p mockforge-cli -- serve --spec examples/openapi-demo.json --http-port 3000

# Full-featured: all protocols + admin UI
MOCKFORGE_LATENCY_ENABLED=true \
MOCKFORGE_FAILURES_ENABLED=false \
MOCKFORGE_WS_REPLAY_FILE=examples/ws-demo.jsonl \
MOCKFORGE_RESPONSE_TEMPLATE_EXPAND=true \
cargo run -p mockforge-cli -- serve \
  --spec examples/openapi-demo.json \
  --http-port 3000 --ws-port 3001 --grpc-port 50051 \
  --admin --admin-port 9080
```

The admin UI is available at `http://localhost:9080` when `--admin` is passed.

## Architecture

Request flow: **OpenAPI spec** → parsed by `mockforge-core` → routes registered → incoming request matched by path/method → response generated (with optional template expansion, latency injection, chaos engineering) → returned to client.

Key architectural boundaries:
- `mockforge-core` owns routing, middleware, and OpenAPI parsing — most protocol crates depend on it
- Each protocol crate (`mockforge-http`, `mockforge-grpc`, etc.) implements its own server/listener
- `mockforge-plugin-core` defines the plugin trait; `mockforge-plugin-loader` dynamically loads `.so`/`.dylib` plugins
- `mockforge-bench` generates k6 scripts from OpenAPI specs via Handlebars templates

## Project Structure

### Foundation Layer
| Crate | Purpose |
|-------|---------|
| `mockforge-core` | Core server, routing, OpenAPI parsing, middleware |
| `mockforge-data` | Data models, serialization, storage |
| `mockforge-template-expansion` | Handlebars template rendering |
| `mockforge-schema` | Schema validation |
| `mockforge-world-state` | Stateful mock behavior |

### Protocol Layer
| Crate | Purpose |
|-------|---------|
| `mockforge-http` | HTTP protocol (primary protocol) |
| `mockforge-grpc` | gRPC protocol support |
| `mockforge-ws` | WebSocket protocol |
| `mockforge-graphql` | GraphQL protocol |
| `mockforge-kafka` | Kafka protocol |
| `mockforge-mqtt` | MQTT protocol |
| `mockforge-amqp` | AMQP protocol |
| `mockforge-smtp` | SMTP protocol |
| `mockforge-ftp` | FTP protocol |
| `mockforge-tcp` | TCP protocol |

### Plugin & Extension Layer
| Crate | Purpose |
|-------|---------|
| `mockforge-plugin-core` | Plugin trait definitions |
| `mockforge-plugin-sdk` | SDK for plugin authors |
| `mockforge-plugin-loader` | Dynamic plugin loading |
| `mockforge-plugin-registry` | Plugin marketplace |
| `mockforge-plugin-cli` | Plugin CLI management |

### Bench, Test & Observability
| Crate | Purpose |
|-------|---------|
| `mockforge-bench` | k6 script generation, WAFBench, OWASP testing |
| `mockforge-test` | Test framework and utilities |
| `mockforge-observability` | Observability infrastructure |
| `mockforge-tracing` | Distributed tracing |
| `mockforge-analytics` | Usage analytics |
| `mockforge-reporting` | Report generation |
| `mockforge-chaos` | Chaos engineering |
| `mockforge-performance` | Performance profiling |
| `mockforge-pipelines` | Request/response pipelines |
| `mockforge-recorder` | Traffic recording |
| `mockforge-route-chaos` | Per-route chaos injection |
| `mockforge-scenarios` | Test scenario management |

### User-Facing
| Crate | Purpose |
|-------|---------|
| `mockforge-cli` | Main CLI binary |
| `mockforge-ui` | React admin UI (Vite + React 19 + TypeScript) |
| `mockforge-sdk` | Client SDK |
| `mockforge-registry-server` | Multi-tenant registry API |

### Infrastructure
| Crate | Purpose |
|-------|---------|
| `mockforge-tunnel` | Tunnel support |
| `mockforge-vbr` | Virtual branch resilience |
| `mockforge-collab` | Collaborative features (SQLite/SQLx) |
| `mockforge-k8s-operator` | Kubernetes operator |
| `mockforge-runtime-daemon` | Runtime daemon service |
| `mockforge-federation` | Federation support |

### Examples & Tests
| Crate | Purpose |
|-------|---------|
| `examples/plugins/response-graphql` | Example GraphQL plugin |
| `examples/test-integration` | Integration test examples |
| `test_openapi_demo` | OpenAPI demo test fixture |
| `desktop-app` | Desktop application (Tauri) |
| `tests` | Workspace-level integration tests |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SaaSy-Solutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
