---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Development Commands

**Build System (Makefile-driven):**
```bash
make dev          # Clean build from scratch: proto + build
make build        # Build all binaries (server, agent) to /bin
make proto        # Generate gRPC code using Buf (buf generate)
make pack         # Generate pack artifacts via protoc-gen-croupier
make test         # Run unit tests with race detection
make clean        # Remove build artifacts and generated code
```

**Local Development Setup:**
```bash
git clone --recursive https://github.com/cuihairu/croupier.git
go mod download && make submodules
./scripts/dev-certs.sh    # Generate self-signed TLS certs
buf lint && buf generate  # Generate proto code
make build               # Build binaries
```

**Testing:**
```bash
make test                 # All tests with race detection
go test ./internal/...    # Subset testing
./bin/croupier-server --config configs/server.yaml validate      # Config validation
```

**Code Style:**
```bash
gofmt -w .                # Format all Go files
gofmt -l .                # List files that need formatting
```

**IMPORTANT: Before committing any changes, ALWAYS run `gofmt -w .` to ensure all Go files are properly formatted.**

## Architecture Overview

Croupier implements a **three-tier distributed GM backend system**:

1. **Permission Control Layer** - RBAC/ABAC system independent of game logic
2. **Game Control Layer** - Function registration-driven game operations
3. **Observable Display Layer** - Descriptor-driven UI generation

### Core Components

**Server** (`internal/server/`)
- Central control plane with gRPC (8443) + HTTP REST (18780)
- Two main services: `ControlService` (agent registration) and `FunctionService` (invocation routing)
- Features: load balancing, RBAC, audit chain, approval workflows, multi-game scoping

**Agent** (`internal/agent/`)
- Distributed proxy in game networks, outbound mTLS to Server
- Local gRPC listener (19090) for game server function registration
- Bidirectional tunnel support for request/response multiplexing
- Job execution with async streaming, idempotency, cancellation

### Data Flow Pattern
```
Web UI → Server (HTTP) → Load Balancer → Agent → Game Server
```

## Key Development Patterns

**Protocol-First Development:**
- All APIs defined in `proto/` using Buf toolchain
- Custom protoc plugin (`protoc-gen-croupier`) generates pack artifacts
- Generated code in `gen/` (ignored in git)

**CRITICAL: Protobuf Code Generation**
- **ALWAYS** use `make proto` to generate protobuf code
- Buf uses **remote plugins** with fixed versions (e.g., buf.build/protocolbuffers/go:v1.36.11)
- **NEVER** use local `protoc` directly - it will generate incompatible code
- Local protoc version mismatches will cause compilation failures
- Buf config (`buf.gen.yaml`) specifies exact plugin versions to match CI environment

**Descriptor-Driven Architecture:**
- Functions defined via protobuf + JSON Schema descriptors
- UI auto-generates forms, validation, and permission checks from single source
- Function packs (`.tgz`) bundle descriptors, schemas, and UI plugins

**Configuration Management:**
- Multi-layer: YAML → includes → profiles → env vars → CLI flags
- Environment prefixes: `CROUPIER_SERVER_*`, `CROUPIER_AGENT_*`
- Config validation: `./croupier config test`

**Idempotency & Task Model:**
- All operations support `idempotency-key` to prevent duplicate side effects
- Async tasks with event streaming (progress/logs/done/error)
- Task cancellation via `CancelTask` RPC

**Build Tags for Features:**
- `pg` tag: PostgreSQL support for approvals
- `sqlite` tag: SQLite approvals store
- Enables flexible deployment options

## Project Structure Essentials

```
cmd/                      # Binary entry points (server, agent, unified CLI)
proto/                    # Protobuf definitions (Buf workspace)
internal/server/          # Server business logic (control, function, http, registry)
internal/agent/           # Agent logic (tunnel, local server, jobs)
internal/auth/            # RBAC, JWT, TOTP, user management
internal/function/        # Descriptor loading and validation
internal/jobs/            # Job state machine and execution
internal/loadbalancer/    # Load balancing strategies (RR, consistent hash, least conn)
sdks/                     # Multi-language SDKs (submodules: go, cpp, java)
web/                      # Frontend submodule (Umi Max + Ant Design)
configs/                  # Configuration templates and examples
examples/                 # Demo game servers and invokers
```

## Important Implementation Details

**Security Architecture:**
- Enforced mTLS for all inter-service communication
- Field-level masking for sensitive data in audit logs
- Two-person rule enforcement for high-risk operations
- Audit chain with hash-based integrity

**Multi-Game Scoping:**
- All operations scoped by `game_id`/`env` for tenant isolation
- Registry indexed by `(game_id, function_id)` for function routing
- HTTP headers `X-Game-ID`/`X-Env` propagated through call chain

**Load Balancing Abstraction:**
- Strategy interface with multiple implementations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cuihairu/croupier](https://github.com/cuihairu/croupier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
