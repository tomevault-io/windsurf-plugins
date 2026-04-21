---
trigger: always_on
description: Linnix is an eBPF-powered Linux observability platform that captures process lifecycle events (fork/exec/exit) with lightweight CPU/memory telemetry and streams them to user-space for AI-powered incident detection.
---

# Linnix AI Agent Instructions

## Architecture Overview

Linnix is an eBPF-powered Linux observability platform that captures process lifecycle events (fork/exec/exit) with lightweight CPU/memory telemetry and streams them to user-space for AI-powered incident detection.

**Core Components:**
- **cognitod** (`cognitod/`) - Main daemon that loads eBPF programs, consumes perf buffers, maintains process state, and exposes HTTP/SSE APIs
- **linnix-ai-ebpf** (`linnix-ai-ebpf/`) - Dual-space eBPF collector: kernel-side tracepoints/kprobes and userland Aya bindings
- **linnix-cli** (`linnix-cli/`) - CLI client consuming SSE streams from cognitod
- **linnix-reasoner** (`linnix-reasoner/`) - Fetches system snapshots from cognitod and sends to OpenAI LLM for semantic analysis
- **insight_tool** (Python, `insight_tool/`) - Dataset pipeline for incident→insight training data; validates against JSON Schema

**Data Flow:**
```
Kernel (eBPF probes) → Perf buffers → cognitod runtime → Handler pipeline (rules/JSONL/ILM) → HTTP/SSE endpoints → CLI/Dashboard/Reasoner
```

## Critical Developer Workflows

### Build & Test
```bash
# Full workspace build
cargo build --release

# Build eBPF artifacts (requires nightly-2024-12-10)
cargo xtask build-ebpf

# Test suite (uses nextest)
make test  # or RUSTFLAGS="-D warnings" cargo nextest run --workspace

# Format + lint (CI enforces this)
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo deny check
```

### Running Cognitod Locally
Cognitod requires root/CAP_BPF to load eBPF programs. The daemon looks for compiled eBPF objects in these paths (priority order):
1. `LINNIX_BPF_PATH` env var
2. `/usr/local/share/linnix/linnix-ai-ebpf-ebpf`
3. `target/bpfel-unknown-none/release/linnix-ai-ebpf-ebpf`
4. Fallback search in `target/bpf/*.o`

```bash
# Build workspace + eBPF, then run daemon
cargo build --release -p cognitod
cargo xtask build-ebpf
sudo LINNIX_BPF_PATH=target/bpf/linnix-ai-ebpf-ebpf.o \
  ./target/release/cognitod --config configs/linnix.toml --handler rules:configs/rules.yaml
```

Use `demo_phase1_local.sh` for a complete local demo with synthetic events.

### BTF-Dependent Features
Cognitod uses kernel BTF (`/sys/kernel/btf/vmlinux`) to dynamically derive struct offsets for:
- `task_struct` fields (real_parent, tgid, se.sum_exec_runtime)
- RSS accounting (either `mm_struct->rss_stat` or `signal_struct->rss_stat` depending on kernel version)

Without BTF, the daemon runs in degraded mode (no per-process CPU/RSS metrics). See `cognitod/src/bpf_config.rs::derive_telemetry_config()`.

### Handler System
Cognitod supports pluggable event handlers specified via `--handler <type>:<path>`:
- `jsonl:<path>` - Append events/snapshots as JSONL (see `cognitod/src/handler/mod.rs::JsonlHandler`)
- `rules:<path>` - YAML-based rule engine for fork storms, exec rates, subtree CPU spikes (see `cognitod/src/handler/local_ilm/`)
- Handlers implement the `Handler` trait (`async fn on_event`, `async fn on_snapshot`)

## Project-Specific Conventions

### eBPF/Userspace Boundary
- Kernel events use `ProcessEventWire` (POD struct defined in `linnix-ai-ebpf-common`)
- Userspace extends to `ProcessEvent` with tags, hash caches, and helper methods
- CPU/mem percentages stored as `u16` milli-percent (10000 = 100.0%); `PERCENT_MILLI_UNKNOWN` (65535) indicates no data

### Workspace Structure
- Multi-crate workspace with shared dependencies in root `Cargo.toml` under `[workspace.dependencies]`
- eBPF code lives in `linnix-ai-ebpf/linnix-ai-ebpf-ebpf/src/program.rs`; uses Aya's attribute macros (`#[tracepoint]`, `#[kprobe]`)
- Tests use nextest with profiles: `default` (1 retry for flaky tests), `e2e` (serial execution)

### Dual Licensing
- Core daemon/CLI: **AGPL-3.0** or commercial
- eBPF code: **GPL-2.0** or MIT (eBPF programs must be GPL-compatible)
- Sign CLA before contributing (see `CONTRIBUTING.md`)

## Key Integration Points

### External Dependencies
- **Aya** (`git` dependency pinned to `fe8e1c48`) - eBPF loading/management; not published to crates.io
- **BTF library** (`btf` crate) - Parse `/sys/kernel/btf/vmlinux` for dynamic offset resolution
- **Tokio** - Async runtime; perf buffer polling uses `tokio::spawn` workers per CPU
- **Axum** - HTTP server exposing `/events` (SSE), `/processes`, `/graph/{pid}`, `/insights`, `/metrics`, etc.

### API Endpoints (see `cognitod/src/api/mod.rs`)
- `GET /stream` - Server-sent events for real-time process events
- `GET /processes` - All live processes
- `GET /graph/{pid}` - Process ancestry graph (descendants + lineage)
- `GET /insights` - Recent insights from rule engine or LLM
- `GET /metrics` - Prometheus-compatible metrics

### Configuration
Cognitod loads TOML config from:
1. `LINNIX_CONFIG` env var
2. `--config` CLI flag
3. `/etc/linnix/linnix.toml` (default)

Schema in `cognitod/src/config.rs`. Notable fields:
- `runtime.offline = true` disables all external HTTP egress (Slack/PagerDuty/Prometheus)
- `telemetry.sample_interval_ms` controls eBPF sampling frequency

## Code Patterns

### Error Handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linnix-os/linnix](https://github.com/linnix-os/linnix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
