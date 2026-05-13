---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

orb8 is an eBPF-powered observability toolkit for Kubernetes with first-class GPU telemetry. Built entirely in **Rust** using the aya framework, it provides low-overhead monitoring of network flows, system calls, and GPU performance optimized for AI/ML workloads.

**Architecture**: Dual-mode platform supporting both cluster-wide monitoring (DaemonSet) and standalone on-demand tracing.

**Current Status**: Phase 4 (Deploy and Run Anywhere) - Network flows working, test infrastructure in place, documentation overhauled.

## Monorepo Structure

orb8 is organized as a **Cargo workspace** with multiple crates:

```
orb8/
├── Cargo.toml                    # Virtual workspace root (no root package)
├── Dockerfile                    # Multi-stage (CI) and local (fast) build targets
├── orb8-probes/                  # eBPF probes (Rust, kernel space)
├── orb8-common/                  # Shared types between kernel/user space
├── orb8-agent/                   # Node agent (DaemonSet)
├── orb8-server/                  # Central API server (stub)
├── orb8-cli/                     # CLI tool
├── orb8-proto/                   # gRPC protocol definitions
├── deploy/                       # K8s manifests (DaemonSet, RBAC, kind config, test pods)
├── scripts/                      # Dev setup, smoke-test.sh, e2e-test.sh
└── docs/
    ├── ARCHITECTURE.md           # Detailed technical design
    └── ROADMAP.md                # Phase-based implementation plan
```

### Workspace Commands

```bash
# Build all crates (excludes orb8-probes on non-Linux)
cargo build

# Build specific crate
cargo build -p orb8-agent

# Test (uses default-members, excludes orb8-probes which is #![no_main])
cargo test

# NEVER use cargo test --workspace (orb8-probes will fail linking)

# Run specific binary
cargo run -p orb8-cli -- --help
```

## Build and Development Commands

### Quick Start

```bash
make magic          # Build, test, install (uses VM on macOS, native on Linux)
make magic-local    # Build, test, install locally without VM
```

### Development Workflow

#### macOS (uses Lima VM for eBPF support)

```bash
make dev            # Setup/start Lima VM (first run: 5-10 min)
make shell          # Enter VM
make status         # Check VM status
make stop           # Stop VM
make clean          # Delete VM completely
```

#### Inside VM or on Linux

```bash
# Build
cargo build                              # Debug build
cargo build --release                    # Release build
cargo build -p orb8-probes              # Build eBPF probes only

# Test
cargo test                              # All tests (default-members)
cargo test --lib                        # Unit tests only
cargo test -p orb8-agent                # Test specific crate

# Run
cargo run -p orb8-cli -- --help
sudo cargo run -p orb8-agent            # Requires root for eBPF
```

### Code Quality

```bash
cargo fmt                               # Format all code
cargo fmt -p orb8-agent                # Format specific crate
cargo clippy --workspace -- -D warnings # Lint (must pass with zero warnings)
cargo check --workspace                 # Type check without building
```

### eBPF Probe Development

eBPF probes are written in Rust using aya-bpf:

```bash
# Build probes (automatically triggered by workspace build)
cargo build -p orb8-probes

# Verify probe compilation (note: no .o extension)
ls target/bpfel-unknown-none/release/orb8_probes

# Load and test probe (requires Linux)
sudo cargo run -p orb8-agent
```

**Critical**: Probes are **embedded in the agent binary** at compile time via `include_bytes_aligned!` in `orb8-agent/build.rs`. The Dockerfile does NOT need to copy probe files separately.

## Architecture

orb8 consists of **eBPF probes** (kernel space) and **Rust services** (user space).

### eBPF Probes (orb8-probes/)

**Run in**: Kernel space
**Language**: Rust (no_std) using aya-bpf
**Compile to**: ELF eBPF relocatable (`target/bpfel-unknown-none/release/orb8_probes`)
**Embedded via**: `include_bytes_aligned!` in agent's build.rs

Probes:
- `network_probe.rs` - Network flow tracing (TC classifier, ingress/egress)

Planned (not yet implemented):
- Syscall monitoring (tracepoint) - Phase 8
- GPU telemetry (kprobe/uprobe) - Phase 9

**Key concept**: TC classifiers run in network/softirq context and cannot access cgroup IDs. Pod identification for network events uses **IP-based enrichment** from the Kubernetes API. Future tracepoint probes (syscall monitoring) will have process context and can use cgroup-based identification.

### User-Space Components

#### orb8-agent (DaemonSet)

**Purpose**: Runs on every Kubernetes node

**Responsibilities**:
- Load eBPF probes into kernel
- Poll ring buffers for events
- Watch Kubernetes API for pod metadata
- Map pod IPs → pod metadata (primary enrichment path)
- Map cgroup IDs → pod metadata (for future tracepoint probes)
- Aggregate flow metrics
- Expose gRPC API (:9090)

**Key files**:
- `probe_loader.rs` - Manages eBPF probe lifecycle, ring buffer polling
- `k8s_watcher.rs` - Watches pods, populates IP and cgroup caches

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ignoramuss/orb8](https://github.com/Ignoramuss/orb8) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
