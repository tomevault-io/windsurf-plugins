---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

schedcp is a Linux scheduler optimization project that uses the sched-ext (scx) framework to implement and test various BPF-based kernel schedulers. The project includes multiple scheduler implementations in both C and Rust, along with benchmarking tools, workloads, and an MCP (Model Context Protocol) server for AI-assisted scheduler management.

## Essential Build Commands

### Building the MCP Server and CLI
```bash
# Build the schedcp MCP server and CLI tool
cd mcp
cargo build --release

# Binaries will be available at:
# - mcp/target/release/schedcp (MCP server)
# - mcp/target/release/schedcp-cli (CLI tool)
```

### Building Schedulers
```bash
# Install dependencies first (one-time setup)
cd scheduler
make deps

# Build all schedulers (C and Rust) - outputs to sche_bin/
make

# Build specific types
make build-c        # Build only C schedulers
make build-rust     # Build only Rust schedulers
make build-tools    # Build tools (scx_loader, scxctl, scxtop) - outputs to tools/

# Generate scheduler documentation
make doc

# Clean build artifacts
make clean

# Update scx submodule
make update

# Install schedulers to ~/.schedcp/scxbin/
make install
```

### Running Tests
```bash
# Unit tests for sched-ext framework
cd scheduler/scx
meson setup build
meson test -v -C build

# Run specific benchmark workloads
python workloads/basic/scheduler_test/schbench_bench_start.py
python workloads/llama.cpp/llamacpp_bench_start.py
python workloads/cxl-micro/cxl_micro_bench_start.py
```

## High-Level Architecture

### Core Components Integration

**MCP Server Architecture**: The heart of the project is the MCP server (`mcp/src/`) which provides AI-assisted scheduler management. Key integration points:

- **scheduler_manager.rs**: Manages the lifecycle of kernel schedulers, interfaces with embedded scheduler binaries, handles both built-in and custom schedulers
- **scheduler_generator.rs**: Creates, compiles, and verifies custom BPF schedulers from source code (private module accessed through manager)
- **system_monitor.rs**: Collects real-time CPU, memory, and scheduler metrics from `/proc` for performance analysis
- **workload_profile.rs**: Creates natural language workload descriptions and tracks performance history across different schedulers
- **storage.rs**: Persists workload data in `schedcp_workloads.json`, enabling learning from past optimizations

**Scheduler Framework Integration**: The project embeds all sched-ext schedulers as resources, enabling standalone operation without requiring local builds. The embedded approach allows the MCP server to manage any scheduler configuration programmatically.

**Workload-Scheduler Optimization Loop**: The architecture enables a closed-loop optimization system where:
1. Workloads are described in natural language and classified
2. Multiple schedulers are tested automatically with optimal configurations
3. Performance history is tracked and used for future recommendations
4. AI assistants can make data-driven scheduler selections

### Directory Structure and Data Flow

```
schedcp/
├── mcp/                           # MCP server for AI-assisted optimization
│   ├── src/
│   │   ├── scheduler_manager.rs   # Scheduler lifecycle management
│   │   ├── scheduler_generator.rs # Custom scheduler creation/compilation
│   │   ├── system_monitor.rs      # Real-time metrics collection
│   │   ├── workload_profile.rs    # Workload classification and history
│   │   ├── storage.rs             # Persistent performance data
│   │   ├── main.rs                # MCP server entry point
│   │   ├── lib.rs                 # Core MCP implementation
│   │   └── cli.rs                 # CLI tool implementation
│   ├── new_sched/                 # Custom scheduler working directory
│   │   ├── loader                 # BPF loader for custom schedulers
│   │   └── *.bpf.{c,o}           # Custom scheduler sources and objects
│   └── schedcp_workloads.json     # Performance history database
├── scheduler/                     # Scheduler build system and metadata
│   ├── scx/                       # sched-ext framework (submodule)
│   ├── sche_bin/                  # Compiled scheduler binaries
│   ├── scheduler_runner.py        # Python scheduler interface
│   └── schedulers.json            # Scheduler metadata and capabilities
└── workloads/                     # Benchmark workloads for testing
    ├── basic/schbench/            # Scheduler latency benchmark
    ├── llama.cpp/                 # LLM inference workload
    └── cxl-micro/                 # Memory subsystem benchmark
```

**Critical Integration Points**:
- `schedulers.json` contains metadata about each scheduler's algorithm, tuning parameters, and use cases
- `scheduler_runner.py` provides a Python interface that the MCP server uses to control schedulers
- The MCP server embeds all scheduler binaries, making it self-contained for deployment

### Scheduler Selection and Optimization Logic

The project implements an intelligent scheduler selection system:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eunomia-bpf/schedcp](https://github.com/eunomia-bpf/schedcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
