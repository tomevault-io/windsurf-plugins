---
trigger: always_on
description: **CLAUDE.md Version**: 1.0.0
---

# spinbox

**CLAUDE.md Version**: 1.0.0
**Last Updated**: 2025-01-09
**Last Review**: @claude

---

## Overview

- **Type**: Containerd shim runtime
- **Primary Language**: Go 1.25
- **Architecture**: Host shim + Guest daemon (vminitd) communicating over vsock/TTRPC
- **Tech Stack**: containerd, QEMU/KVM, CNI networking, TTRPC, protobuf

---

## Purpose of This File

This CLAUDE.md is the **authoritative source** for development guidelines. It is read by Claude Code before every interaction and takes priority over user prompts.

**Hierarchy**:
- This file contains universal project rules
- Subdirectories contain specialized CLAUDE.md files that extend these rules
- See [Directory-Specific Context](#directory-specific-context) for links

**Maintenance**: Update this file as patterns evolve. Use `#` during sessions to add memories organically.

---

## Universal Development Rules

### Code Quality [MUST]

- **MUST** check all errors - no `_ = err` without justification comment
- **MUST** use `context.Context` for cancellation and timeouts in all blocking operations
- **MUST** run `task lint` before committing (enforces 50+ linter rules)
- **MUST** include tests for new functionality (target: >80% coverage)
- **MUST NOT** commit secrets, API keys, or credentials
- **MUST NOT** edit generated files (`*.pb.go`, `*_ttrpc.pb.go`) - regenerate instead

### Go Best Practices [SHOULD]

- **SHOULD** use structured logging via `containerd/log` package
- **SHOULD** prefer interfaces for dependencies (easier testing)
- **SHOULD** use `errors.Is()` and `errors.As()` for error checking
- **SHOULD** wrap errors with context: `fmt.Errorf("operation: %w", err)`
- **SHOULD** use `sync.Mutex` with clear lock ordering to prevent deadlocks
- **SHOULD** document lock ordering in package comments (see `internal/shim/task/service.go`)

### Anti-Patterns [MUST NOT]

- **MUST NOT** ignore errors: `_, err := doThing(); return nil` is forbidden
- **MUST NOT** use `panic()` for error handling (use error returns)
- **MUST NOT** use `unsafe` package without explaining why in comment
- **MUST NOT** hold locks during slow operations (network calls, VM operations)
- **MUST NOT** mutate global state without synchronization

### Concurrency Patterns [CRITICAL]

This project uses sophisticated concurrency. Follow these patterns:

- **State machines** for lifecycle management (see `internal/shim/lifecycle/state.go`)
- **Collect-then-execute** pattern: gather resources under lock, release lock, then operate
- **Atomics** for lock-free counters and flags (`sync/atomic`)
- **Channel ownership**: document producers and consumers in comments

---

## Core Development Commands

### Quick Reference

```bash
# Development
task build:shim          # Build containerd shim (static, Linux x86_64)
task build:vminitd       # Build guest init daemon
task build:initrd        # Build initrd image (requires Docker)
task build:kernel        # Build VM kernel (requires Docker)
task build               # Build all components

# Testing
go test ./...                                    # Unit tests
go test -v -race ./...                           # Unit tests with race detection
go test -v -timeout 10m -tags=integration ./integration/...  # Integration (requires KVM)

# Validation
task lint                # Run golangci-lint (50+ linters)
task validate            # Run all validation checks
task verify:vendor       # Verify go.mod/go.sum are up to date

# Code Generation
task protos              # Generate protobuf files
task check:protos        # Verify protobufs are up to date

# Cleanup
task clean               # Clean build artifacts
```

### Pre-Commit Checklist [CRITICAL]

Run these commands before every commit:

```bash
go fmt ./... && \
goimports -w . && \
task lint && \
go test -race ./...
```

All checks must pass before committing.

---

## Project Structure

### Root Layout

```
.
├── cmd/                           # Application entry points
│   ├── containerd-shim-spinbox-v1/  # Host: containerd shim
│   └── vminitd/                     # Guest: PID 1 inside VM
├── internal/                      # Private application code
│   ├── shim/                        # Shim-side implementation
│   │   ├── task/                    # Task service (Create, Start, Delete)
│   │   ├── lifecycle/               # State machine and cleanup
│   │   ├── cpuhotplug/              # CPU hotplug controller
│   │   └── memhotplug/              # Memory hotplug controller
│   ├── guest/                       # Guest-side implementation
│   │   └── vminit/                  # VM init daemon
│   ├── host/                        # Host-side services
│   │   ├── vm/qemu/                 # QEMU VM lifecycle
│   │   └── network/                 # CNI networking
│   └── config/                      # Configuration loading
├── api/                           # Protobuf definitions
│   └── services/                    # TTRPC service definitions
├── integration/                   # Integration tests
├── docs/                          # Documentation
└── examples/                      # Example configurations
```

### Key Directories

**Applications** (`cmd/`):
- **`cmd/containerd-shim-spinbox-v1/`** - Shim entry point
  - Loads config from `/etc/spinbox/config.json` or `SPINBOX_CONFIG` env var

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spin-stack/spinbox](https://github.com/spin-stack/spinbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
