---
trigger: always_on
description: **Claude AI Assistant Guide** - Updated Feb 22, 2026
---

# hyprstream Development Guide

**Claude AI Assistant Guide** - Updated Feb 22, 2026

## Core Philosophy

- **Models are Git repositories** — version-controlled via git2db
- **Adapters are files** — stored in `model/adapters/` as `.safetensors` (NOT branch-based)
- **git2db handles all Git operations** — no custom git wrappers, no raw `Repository::open()`
- **Storage drivers optimize disk** — overlay2 on Linux (~80% savings)

## Status

**Production**: PyTorch inference (CPU/CUDA/ROCm), git2db model management, file-based LoRA adapters, OpenAI-compatible REST API, UTF-8 streaming
**Experimental**: XET large file storage (enabled by default), Test-Time Training (TTT) with per-tenant LoRA deltas

## Build

```bash
export LIBTORCH=/path/to/libtorch
export LD_LIBRARY_PATH=$LIBTORCH/lib:$LD_LIBRARY_PATH
cargo build --release                    # Standard (includes xet, gittorrent, systemd, otel)
cargo build --features cuda --release    # CUDA marker (backend from tch-rs/libtorch)
cargo build --features bnb --release     # bitsandbytes quantization
cargo build --features overlayfs --release
cargo build --no-default-features --features "gittorrent,xet,otel" --release  # No systemd
cargo test --workspace --release
```

**Feature flags**: `default = [gittorrent, xet, systemd, otel]`, `cuda` (empty marker), `bnb`, `overlayfs`, `download-libtorch`, `experimental`
**Backend**: CPU/CUDA/ROCm controlled by tch-rs dependency (fork: github.com/hyprstream/tch-rs branch: hip), NOT cargo features

## Crate Map

| Crate | Purpose |
|-------|---------|
| `hyprstream` | Main app: runtime, storage, git, training, API, services, CLI (MIT OR AGPL-3.0) |
| `git2db` | Git repository management library (has own CLAUDE.md) |
| `git-xet-filter` | XET/LFS large file storage (libgit2 filter) |
| `gittorrent` | P2P model distribution |
| `cas-serve` | Content-addressable storage server |
| `hyprstream-flight` | Arrow Flight SQL server |
| `hyprstream-metrics` | Metrics (DuckDB/DataFusion) |
| `hyprstream-rpc` | Cap'n Proto RPC: ZMQ transport, crypto, signed envelopes, JWT auth |
| `hyprstream-rpc-derive` | Proc macros: `ToCapnp`, `FromCapnp`, `#[authorize]`, `#[service_factory]`, `generate_rpc_service!` |
| `hyprstream-rpc-build` | CGR → JSON metadata extraction for macros |
| `hyprstream-workers` | Kata-based worker isolation |
| `hyprstream-containedfs` | Contained filesystem ops |
| `hyprstream-vfs` | Plan 9-inspired VFS namespace multiplexer |
| `hyprstream-tcl` | Tcl (molt) shell for VFS namespace |
| `bitsandbytes-sys` | bitsandbytes FFI bindings |

## Key Source Layout (`crates/hyprstream/src/`)

- `runtime/` — PyTorch engine (`torch_engine.rs`), architectures, model_factory, kv_cache, sampling, templates, LoRA
- `storage/` — Model storage (`model_ref.rs`, `paths.rs`), `adapter_manager.rs` (file-based adapters)
- `git/` — Thin wrapper over git2db + `helpers.rs` (tag creation etc.)
- `training/` — TTT (`ttt.rs`), per-tenant deltas (`tenant_delta.rs`, `delta_pool.rs`), merge strategies, quality filter, checkpoints
- `api/` — OpenAI compat types, tool calling
- `server/` — Axum routes (`/v1/models` lists worktrees as `model:branch`), state management
- `services/` — RPC service implementations + `factories.rs` (inventory-based auto-discovery)
- `schema/` — Cap'n Proto schemas: `registry.capnp`, `inference.capnp`, `model.capnp`, `policy.capnp`, `worker.capnp`, `mcp.capnp`

## Services (10 total, registered via `#[service_factory]` in `services/factories.rs`)

**ZMQ RPC** (Cap'n Proto over ZMQ, REQ/REP): registry, model, policy, worker, mcp
**HTTP** (Axum): oai (OpenAI-compat), oauth (OAuth 2.1), flight (Arrow Flight SQL)
**Proxies** (ZMQ): streams (PULL→XPUB), event (XSUB→XPUB)

**Security**: CURVE transport encryption → Ed25519 signed envelopes → Casbin + JWT authorization
**Spawner modes**: Tokio (async), Thread (!Send types like tch-rs), Subprocess (systemd/standalone)
**Endpoints**: Inproc (daemon, zero-copy) or IPC (systemd socket activation)

## Key Patterns

**Git operations** — always use git2db: `registry.repo(&id)?.branch().create(...)`, `handle.staging().add_all()?`
**Adapters** — `AdapterManager::new(path)`, files in `adapters/` as `00_name.safetensors`
**Tags** — `crate::git::helpers::create_tag(path, "v1.0")?`
**Generation** — `engine.generate(req)?` returns `TextStream` (futures::Stream), handles UTF-8 internally via `DecodeStream`
**TTT** — `TestTimeTrainer::adapt_tenant()` for inference-time adaptation, `DeltaPool` for per-tenant LRU management
**XET/LFS** — Filter initialized in main.rs via `git2db::xet_filter::initialize()`, auto-smudges on git ops; fallback: `git2db::LfsStorage`

## RPC & Security

### Code Generation Pipeline

```
.capnp schema → capnpc → CGR binary → hyprstream-rpc-build → JSON metadata
                                                                    ↓
                                          generate_rpc_service! macro → typed clients + handlers + dispatch
```

### Security Model (3 layers)

1. **Transport**: CURVE encryption (Curve25519) on TCP sockets via `CurveConfig`
2. **Application**: Ed25519 signed `SignedEnvelope` — survives message forwarding
3. **Authorization**: Casbin policy + JWT scopes (`action:resource:identifier`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyprstream/hyprstream](https://github.com/hyprstream/hyprstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
