---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

K8E is a CNCF-conformant Kubernetes distribution packaged as a single binary under 100MB, purpose-built for secure, isolated AI agent execution at scale. It provides built-in sandbox orchestration (warm pools, session management), a gRPC gateway for sandbox operations, a CLI command group (`k8e sandbox`) bridging AI agents to sandbox infrastructure, and client SDKs in Python and TypeScript.

## Architecture (Big Picture)

The repository is organized as a **Zig-based Go project**:

```
.
├── build.zig                      # Zig build definitions (Go compilation, cross-compilation)
├── main.go                        # CLI entry point — registers all commands
├── cmd/                           # Individual command entry points
│   ├── server/                    # Control plane + optional agent
│   ├── agent/                     # Agent-only (kubelet + containerd)
│   ├── kubectl/                   # kubectl wrapper
│   └── sandbox-gateway.go         # Sandbox gRPC gateway (cli/cmds)
├── pkg/
│   ├── cli/cmds/                  # CLI flag definitions + command wiring
│   │   ├── root.go                # App setup, global flags
│   │   ├── server.go              # Server struct + all server flags
│   │   ├── agent.go               # Agent struct + all agent flags
│   │   ├── sandbox.go             # sandbox CLI command group
│   │   └── sandbox_gateway.go     # sandbox-gateway command
│   ├── server/                    # Server daemon orchestration
│   ├── agent/                     # Agent daemon orchestration
│   ├── daemons/                   # Individual daemon implementations
│   ├── sandboxmatrix/             # Core sandbox orchestration
│   │   ├── controller.go          # Warm pool reconciler, session GC
│   │   ├── api/v1alpha1/          # CRD types (SandboxMatrix, SandboxWarmPool, SandboxSession)
│   │   └── grpc/
│   │       ├── server.go          # gRPC SandboxService (create/destroy/exec sessions)
│   │       ├── orchestrator.go    # Orchestration logic (sub-agents, confirm actions)
│   │       └── pb/                # Generated protobuf Go code
│   ├── sandbox/client/           # gRPC client + skill installation
│   │   ├── client.go              # gRPC client with TLS auto-discovery
│   │   └── install.go             # Skill installation for claude code/codex/pi
│   ├── sandboxcli/                # CLI command handlers for k8e sandbox
│   │   ├── commands.go            # 10 sandbox command handlers
│   │   ├── session.go             # Session state persistence + flock locking
│   │   ├── snapshot.go            # Workspace snapshot save/restore
│   │   └── manifest.go            # Declarative workspace manifest
│   ├── sandboxmatrix/grpc/        # gRPC gateway — proxies exec/file ops to sandboxd pods
│   ├── configfilearg/             # Config file argument parsing
│   ├── deploy/                    # Kubernetes manifests and Helm charts
│   ├── apis/                      # Internal API types
│   ├── bootstrap/                 # Cluster bootstrap (token generation)
│   ├── token/                     # Token management
│   └── ...                        # certmonitor, secretsencrypt, cgroups, vpn, etc.
├── proto/sandbox/v1/              # Protobuf definitions for sandbox gRPC service
├── sandbox/                       # Sandbox container runtime shim (sandboxd)
├── sandboxd/                      # Runtime daemon in Zig (exec, files, networking)
│── skills/k8e-sandbox/            # SKILL.md for agent CLI integration
└── tests/unit.go                  # Test helper utilities
```

### Key Architectural Flows

**Agent submitting work via CLI:**
```
AI Agent (shell command)
  → k8e sandbox run "code" (direct gRPC with TLS)
    → sandbox-grpc-gateway:50051 (TLS gRPC)
      → Orchestrator (K8s API: create/destroy pods)
      → sandboxd HTTP proxy (port 2024 inside sandbox pods)
        → Isolated container (gVisor/Kata/Firecracker)
```

**Direct SDK usage:**
```
Python/TypeScript SDK
  → gRPC SandboxServiceClient (direct, no MCP overhead)
    → sandbox-grpc-gateway
      → Same path as above
```

## Building

```bash
# Full build (simulator + all targets)
make            # or: zig build all

# Build k8e binary only
make k8e        # or: zig build k8e

# Regenerate protobuf/generated code
make generate   # or: zig build generate

# Format code
make format     # or: zig build fmt

# Clean build artifacts
make clean      # or: zig build clean

# Package for distribution
make package          # Full package
make package-cli      # CLI only
make package-airgap   # Airgap package
```

Build outputs go to `./zig-out/`.

## Running & Testing

```bash
# Run the full test suite
make test       # or: zig build test

# Run a specific Go test (examples)
go test ./pkg/server/... -run TestServerStart -v -count=1
go test ./pkg/sandboxmatrix/... -v -count=1
go test ./pkg/sandbox/client/... -v -count=1
go test ./tests/... -v -count=1 -timeout 120s

# Verify compilation of a single package
go build ./pkg/sandboxmatrix/
go vet ./pkg/sandbox/client/
```

Integration tests require a running K8E cluster and are invoked via `make test`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaods/k8e](https://github.com/xiaods/k8e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
