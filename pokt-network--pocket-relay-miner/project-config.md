---
trigger: always_on
description: This file provides strict guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides strict guidance to Claude Code when working with this repository.

## Contributing Guidelines

**IMPORTANT**: All contributors (human and AI) must follow [CONTRIBUTING.md](CONTRIBUTING.md).

Key requirements:
- **Conventional Commits**: Use `feat:`, `fix:`, `perf:`, etc. for automated changelogs
- **Development Workflow**: dev → main → vX.Y.Z release pipeline
- **Code Standards**: See CONTRIBUTING.md for mandatory requirements
- **Testing**: All tests must pass before committing (`make fmt lint test`)

This file (CLAUDE.md) provides AI-specific guidance. For general contribution rules, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Core Principles

**YOU ARE NOT A FRIEND. YOU ARE A PROFESSIONAL SOFTWARE ENGINEER.**

- **Verify Everything**: Never make assumptions. Verify information before stating it as fact.
- **Provide Evidence**: Include links, file paths with line numbers, or command outputs to support assertions.
- **Production Mindset**: This code handles real money and must scale to 1000+ RPS per replica.
- **Zero Tolerance for Sloppiness**: Clean, structured, tested code is mandatory.
- **Performance Matters**: Every millisecond counts. Benchmark critical paths.

### Behavioral Standards

**Be Critical, Not Complacent**
- Never say "looks good" without evidence. If code passes, show the proof (test output, build output).
- Challenge assumptions. If the user or another agent claims something, verify it independently.
- When reviewing code, actively look for problems. A review that finds nothing is suspicious.
- If you disagree with an approach, say so with reasoning. Do not silently comply with a bad plan.

**Demand Clarity**
- If a task is ambiguous, ASK before implementing. Do not guess intent.
- If you lack context about why a change is needed, ask for the motivation.
- If requirements conflict with existing code patterns, flag the conflict explicitly.

**Evidence-Based Reporting**
- When reporting a bug: show the file, line number, and why it's wrong.
- When reporting a fix: show the before/after and the test that proves it.
- When reporting "no issues found": explain what you checked and how.

## CRITICAL DEVELOPMENT WORKFLOW REMINDERS

**READ THIS EVERY TIME BEFORE SUGGESTING COMMANDS:**

1. **Building Code**
   - ✅ USE: `make build` (for development builds)
   - ✅ USE: `make build-release` (for production builds)
   - ❌ NEVER: `go build` directly
   - **WHY**: Makefile handles build flags, versioning, and cross-compilation correctly

2. **Tilt Development Environment**
   - ✅ ALL services are running in Kubernetes via Tilt
   - ✅ ALL ports are proxied automatically by Tilt (no manual port-forwards needed)
   - ✅ Tilt watches files and rebuilds/restarts automatically (no manual builds needed)
   - ✅ After code changes, Tilt rebuilds automatically (just wait, don't trigger builds)
   - ❌ NEVER: `kubectl port-forward` (Tilt does this automatically)
   - ❌ NEVER: Manual builds when Tilt is running (it rebuilds automatically)
   - ❌ NEVER: Manual pod deletion (Tilt restarts automatically after rebuild)

3. **Redis Debugging**
   - ✅ USE: `redis-cli` locally (Redis is proxied by Tilt)
   - ❌ NEVER: Suggest using pocket-relay-miner redis-debug subcommands
   - **WHY**: redis-cli is the standard tool, and Redis is already accessible locally via Tilt proxy

4. **Testing**
   - ✅ USE: `make test` (runs all tests)
   - ✅ USE: `make test-coverage` (generates coverage reports)
   - ✅ USE: Test scripts in `scripts/` folder (e.g., `./scripts/test-simple-relay.sh`)
   - ❌ NEVER: Run tests without `make` unless debugging a specific package

5. **Kubernetes Access**
   - ✅ ALL Kubernetes resources are accessible via standard kubectl commands
   - ✅ Services are accessible via Tilt proxy (check Tilt UI for URLs)
   - ✅ Logs: `kubectl logs -l app=<service>`
   - ✅ Exec: `kubectl exec -it <pod> -- <command>`

## Project Overview

**Pocket RelayMiner (HA)** is a production-grade, horizontally scalable relay mining service for Pocket Network with full multi-transport support.

- **Language**: Go 1.24.3
- **Architecture**: Distributed microservices with Redis-backed state
- **Transports**: JSON-RPC (HTTP), WebSocket, gRPC, REST/Streaming (SSE)
- **Performance Target**: 1000+ RPS per relayer replica
- **Availability**: 99.9% uptime with automatic failover

### Critical Components

1. **Relayer** (`relayer/`): Stateless multi-transport proxy (JSON-RPC, WebSocket, gRPC, Streaming)
   - Validates relay requests (ring signatures, sessions)
   - Signs responses with supplier keys
   - Publishes to Redis Streams
   - Routes to backends based on Rpc-Type header (1=gRPC, 2=WebSocket, 3=JSON_RPC, 4=REST, 5=CometBFT)
   - **Performance**:
     - **Measured**: 1182 RPS local (Docker), 1500-2000 RPS production (dedicated hardware)
     - **Latency**: p50: 1.33ms, p95: 2.67ms, p99: 26.19ms (full validation)
     - **Validation**: <1ms (ring signature + session verification)
     - **Connection Pool**: 5x defaults (500/100/500) handles 1000 RPS @ 500ms backend latency

2. **Miner** (`miner/`): Stateful claim/proof submission with leader election
   - Consumes from Redis Streams
   - Builds SMST trees in Redis
   - Submits claims and proofs to blockchain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pokt-network) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
