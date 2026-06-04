---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YANET is a high-performance software router built on DPDK. It uses a multi-language approach:

- **C + DPDK**: Dataplane (fast-path packet processing)
- **Go**: Control plane (modules, gateway API)
- **Rust**: CLI tools
- **TypeScript/React**: Web UI

## Build & Test Commands

```bash
# Initial setup
git submodule update --init   # DPDK submodule
meson setup build             # configure C/DPDK build

# Build everything
make all                      # builds dataplane + CLI

# Build individual components
make dataplane                # meson compile -C build
make cli                      # cargo build --release --workspace
cd controlplane && go build ./...
cd web && npm install && npm run build

# Debug/sanitizer builds
make setup-debug              # debug build without sanitizers
make setup-asan               # debug + address/undefined sanitizers

# Run tests
make test                     # Go tests + meson tests (cleans go cache first)
make test-asan                # tests with address sanitizer
make test-tsan                # thread sanitizer (separate build-tsan dir)
make test-functional          # functional tests (requires QEMU/VM)
meson test -C build <name>    # run a single C test by name
go test ./modules/route/...   # run Go tests for a specific module

# Formatting & linting
gofmt -w .                    # Go
clang-format -i <file>        # C
cargo +nightly fmt            # Rust (uses nightly-only options in .rustfmt.toml)
cargo clippy                  # Rust lints
make proto-lint               # protobuf formatting check

# Fuzzing
make fuzz                     # build fuzz targets
make fuzz MODULE=<name>       # run specific fuzzer
```

## Architecture

### Repository Layout

Top-level directories and their roles:

- `dataplane/`     — main C/DPDK binary (`main.c`, `config.c`, `dpdk.c`, `worker.c`, `drivers/`, `unittest/`).
- `controlplane/`  — Go gateway, CGO bindings (`ffi/`), root protos (`ynpb/`), control-plane package (`yncp/`), entrypoints (`cmd/`).
- `modules/`       — packet-processing modules (see Module Structure).
- `devices/`       — device adapters (`plain`, `vlan`); same layout as modules.
- `operators/`     — long-running orchestration daemons (see Operators).
- `lib/`           — C support libraries: `controlplane`, `counters`, `dataplane`, `dataplane_ut`, `errors`, `filter`, `fwstate`, `logging`, `utils`, plus `tests/` and `fuzzing/`.
- `api/`           — public C API headers exposed to control plane (`agent.h`, `config.h`, `counter.h`, `info.h`).
- `bindings/go/`   — root-level Go CGO bindings for the agent/shared-memory agent surface.
- `cli/`           — Rust CLI workspace: `core/` (yanet-cli library), `modules/` (shared CLI subcommands), `Makefile`.
- `common/`        — shared libraries across languages (see Shared Libraries).
- `web/`           — TypeScript/React Web UI.
- `subprojects/dpdk/` — DPDK as a Meson subproject.
- `docs/`, `deploy/`, `debian/`, `etc/` — documentation and packaging.

### Data Flow

```
CLI (Rust) --gRPC--> Gateway (Go) --gRPC--> Module Control Plane (Go) --shared memory--> Dataplane (C/DPDK)
```

The dataplane reads configuration from shared memory and continues working with the last valid config if upper layers fail. Configuration updates are applied atomically.

### Gateway (controlplane/)

A single Go gRPC server that proxies requests to module backends. Modules register themselves with the gateway on startup. The gateway routes by gRPC service name to the correct module backend. Also provides an HTTP-to-gRPC translation layer.

Key packages:

- `cmd/` — binary entrypoints: `yncp-director` (gateway daemon), `bird-adapter` (legacy build of the BIRD adapter).
- `gateway/` — flat package: API gateway server (`gateway.go`, `runner.go`, `registry.go`, `service.go`, `auth_service.go`, `cfg.go`) plus client-side helpers used by built-in services and operators (`registrar.go`, `registration_loop.go`, `tls.go`, `credentials.go`).
- `builtin/` — in-process built-in services: `pipeline`, `inspect`, `function`, `counters`, `logging`. Each implements the structural `gateway.Service` interface and is constructed by the director, then passed to `NewGateway` via `gateway.WithService(...)`.
- `internal/auth/`, `internal/version/`, `internal/xgrpc/` — supporting packages.
- `ffi/` — CGO bindings for shared memory (`shm.go`, `agent.go`, `pipeline.go`).
- `ynpb/` — root protobuf definitions: pipeline, device, counters, inspect, logging, auth, function, gateway, module.
- `yncp/` — control-plane package (`cfg.go`, `director.go` — module registration hub, `version.go`).

### Module Structure

Modules in `modules/` follow one of two layouts. New modules use the
**canonical** form; legacy modules are gradually migrated.

**Canonical** (decap, dscp, forward, route — use as reference):

```
modules/<name>/
  api/               # C library for control plane FFI (controlplane.c/h)
  bindings/go/       # CGO wrapper crate consumed by controlplane
  controlplane/      # Go control plane
    <name>pb/        # Protobuf definitions + generated code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yanet-platform/yanet2](https://github.com/yanet-platform/yanet2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
