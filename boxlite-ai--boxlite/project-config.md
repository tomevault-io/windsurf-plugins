---
trigger: always_on
description: This document provides project context for AI-assisted development with Claude Code.
---

# CLAUDE.md

This document provides project context for AI-assisted development with Claude Code.

## Project Overview

**BoxLite** is an embeddable virtual machine runtime for secure, isolated code execution environments.

**Key Concept:** Think "SQLite for sandboxing" - a lightweight library embedded directly in applications without requiring a daemon or root privileges.

**What it does:**
- Hardware-level VM isolation (KVM on Linux, Hypervisor.framework on macOS)
- Runs OCI containers inside lightweight VMs
- Async-first API for running multiple isolated environments concurrently

**Primary use cases:**
- AI Agent Sandbox - Safe execution environment for untrusted AI-generated code
- Serverless Multi-tenant Runtime - Isolated environments per customer/request
- Regulated Environments - Hardware-level isolation for compliance

**For detailed overview, architecture, and use cases, see:**
- [README.md](./README.md) - Project overview, features, quick start
- [docs/architecture/README.md](./docs/architecture/README.md) - Comprehensive architecture documentation

## Tech Stack

**Core:**
- Rust (async, Tokio runtime)
- libkrun hypervisor (KVM/Hypervisor.framework)
- libcontainer (OCI runtime)
- gRPC (tonic) for host-guest communication

**SDKs:**
- Python (PyO3, Python 3.10+)
- C (FFI, early stage)
- Node.js (napi-rs, Node.js 18+)

**For complete tech stack details, see:**
- [docs/architecture/README.md](./docs/architecture/README.md#tech-stack) - Detailed component breakdown

## Project Structure

**High-level layout:**
```
src/
  boxlite/            # Core runtime (Rust) - 19 modules
  shared/             # Shared types and protocol
  cli/                # CLI binary
  server/             # Distributed server
  ffi/                # FFI layer for SDKs
  guest/              # Guest agent (runs inside VM)
  test-utils/         # Test utilities
  deps/               # Vendored C sys crates
    bubblewrap-sys/
    e2fsprogs-sys/
    libgvproxy-sys/
    libkrun-sys/
sdks/
  python/             # Python SDK (PyO3)
  c/                  # C SDK (FFI)
  node/               # Node.js SDK (napi-rs)
examples/python/      # Python examples (7 categorized subdirectories)
docs/                 # Documentation
scripts/              # Build and setup scripts
```

**Key modules in `src/boxlite/src/`:**
- `runtime/` - BoxliteRuntime, main entry point
- `litebox/` - LiteBox handle, command execution
- `vmm/` - VM manager (libkrun, shim controller)
- `jailer/` - Security isolation (seccomp, sandbox-exec, namespaces)
- `portal/` - Host-guest gRPC communication
- `images/` - OCI image management
- `net/`, `volumes/`, `disk/` - Networking, storage, disks
- `db/` - SQLite persistence

**Runtime data directory (`~/.boxlite/`):**
- `images/` - OCI image cache
- `boxes/` - Per-box data (config, disks)
- `db/` - SQLite databases

**For detailed project structure, see:**
- [CONTRIBUTING.md](./CONTRIBUTING.md#project-structure) - Directory layout and organization
- [docs/architecture/README.md](./docs/architecture/README.md) - Component architecture

## Common Commands

**Quick reference:**
```bash
make setup          # Install dependencies (auto-detects OS)
make dev:python     # Build Python SDK locally
make test           # Run Rust tests
make fmt            # Format code
make dist:python    # Build portable Python wheel
```

**For complete command reference and Makefile targets, see:**
- [Makefile](./Makefile) - All available targets with descriptions (run `make help`)
- [CONTRIBUTING.md](./CONTRIBUTING.md#building-from-source) - Development workflow
- [docs/guides/README.md](./docs/guides/README.md#building-from-source) - Platform-specific build instructions

## Code Style

**Rust:** Follow [docs/development/rust-style.md](./docs/development/rust-style.md) which includes:
- [Microsoft Rust Guidelines](https://microsoft.github.io/rust-guidelines) - external reference
- `cargo fmt` for formatting, `cargo clippy` for linting (enforced in CI)
- Async-first (Tokio runtime)
- Error handling via centralized `BoxliteError` enum
- `Send + Sync` for public types

Key guidelines to internalize:
- **M-PANIC-IS-STOP**: Panics terminate, don't use for error handling
- **M-CONCISE-NAMES**: Avoid "Service", "Manager", "Factory" in type names
- **M-UNSAFE**: Minimize and document all unsafe blocks

**Python:**
- Async/await for all I/O
- Context managers (`async with`) for automatic cleanup
- Type hints encouraged

**For complete code style guidelines, see:**
- [docs/development/rust-style.md](./docs/development/rust-style.md) - Rust style guide with Microsoft guidelines
- [src/shared/src/errors.rs](./src/shared/src/errors.rs) - Error handling patterns

## Workflows

**Development workflow:**
1. Create feature branch from `main`
2. Make changes, add tests, update docs
3. Run `cargo test && make fmt`
4. Open Pull Request
5. Address code review feedback
6. Squash merge to main

**Testing:**
- Rust: `cargo test` (unit + integration)
- Python: `pytest` with real VM integration tests

**For complete workflows, see:**
- [CONTRIBUTING.md](./CONTRIBUTING.md#how-to-contribute) - PR process, testing, release
- [.github/workflows/](../.github/workflows/) - CI/CD pipelines

## Important Notes

### Critical for AI Assistants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boxlite-ai/boxlite](https://github.com/boxlite-ai/boxlite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
