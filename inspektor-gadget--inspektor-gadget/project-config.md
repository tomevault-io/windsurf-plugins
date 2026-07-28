---
trigger: always_on
description: Inspektor Gadget is an eBPF-based tool and framework for data collection and system inspection on Kubernetes clusters and Linux hosts. Gadgets are eBPF programs packaged as OCI images.
---

# Copilot Instructions for Inspektor Gadget

Inspektor Gadget is an eBPF-based tool and framework for data collection and system inspection on Kubernetes clusters and Linux hosts. Gadgets are eBPF programs packaged as OCI images.

## Build & Test Commands

```bash
# Build ig (standalone Linux tracer)
make ig

# Build kubectl-gadget (Kubernetes plugin)
make kubectl-gadget

# Build the gadget container image
make gadget-container

# Build eBPF object files (runs inside Docker)
make ebpf-objects

# Unit tests (requires sudo for eBPF)
make test
# Run a single test package
go test -exec sudo -v ./pkg/operators/ebpf/...
# Run a specific test
go test -exec sudo -v -run TestMyTest ./pkg/operators/ebpf/...

# Integration tests (requires a Kubernetes cluster with IG deployed)
make integration-tests
# Run a specific integration test
INTEGRATION_TESTS_PARAMS="-run TestTraceExec" make integration-tests

# Gadget-specific tests
make unit-test-gadgets
make integration-test-gadgets

# Regenerate testdata (needed when eBPF test fixtures change)
make generate-testdata

# Lint (runs in Docker using golangci-lint)
make lint

# Format eBPF C code (runs in Docker)
make clang-format
```

## Architecture

### Core Pipeline

Gadgets flow through a pipeline: **OCI Image → Operators → DataSources → Output**

- **Gadgets** (`gadgets/`) — OCI images containing eBPF C code (`program.bpf.c`), metadata (`gadget.yaml`), and optional WASM modules (`go/program.go`). Built with `ig image build`.
- **Operators** (`pkg/operators/`) — Pluggable stages that process gadget data. Two types:
  - `ImageOperator` — handles OCI media types (e.g., `ebpf/` loads eBPF programs, `wasm/` runs WASM modules)
  - `DataOperator` — transforms data in the pipeline (e.g., `filter/`, `sort/`, `formatters/`, enrichment operators like `kubemanager/`, `socketenricher/`)
  - Operators register via `RegisterOperatorForMediaType()` or `RegisterDataOperator()` in `pkg/operators/registry.go`
  - `DataOperator.Priority()` controls execution order
- **DataSources** (`pkg/datasource/`) — Typed event streams that operators produce and consume. Fields carry annotations for column display, JSON serialization, etc.
- **GadgetContext** (`pkg/gadget-context/`) — Orchestrates a gadget run: loads the OCI image, instantiates operators, wires up data sources, manages lifecycle.
- **Runtimes** (`pkg/runtime/`) — `local/` runs gadgets on the host; `grpc/` runs them remotely via gRPC.

### Three Binaries

| Binary | Entry point | Purpose |
|---|---|---|
| `ig` | `cmd/ig/` | Standalone Linux tracer (no Kubernetes) |
| `kubectl-gadget` | `cmd/kubectl-gadget/` | Kubernetes kubectl plugin |
| `gadgetctl` | `cmd/gadgetctl/` | Remote client (macOS/Windows) |

`kubectl-gadget` and `gadgetctl` are built with `-tags withoutebpf` since they don't run eBPF locally.

### Gadget Structure

Each gadget in `gadgets/` follows this layout:

```
gadgets/trace_open/
  program.bpf.c      # eBPF C source (GPL-2.0, includes <gadget/macros.h> etc.)
  gadget.yaml         # Metadata: datasources, field annotations, params
  go/program.go       # Optional WASM post-processing module (Go or Rust)
  test/               # Gadget-specific tests
  README.md           # Short description with link to full docs
  README.mdx          # Full documentation in MDX format for the website
```

- `gadget.yaml` defines datasources with field annotations (`columns.width`, `columns.hidden`, `columns.alignment`) and operator params.
- New gadgets must be added to `gadgets/Makefile` and have symlinks created in `docs/gadgets` to the `README.mdx`.

### WASM Extension API

Gadgets can include WASM modules for post-processing. The API is in `wasmapi/go/` (Go) and `wasmapi/rust/` (Rust), using `//go:wasmimport` host function bindings for datasource access, field manipulation, and event emission.

### Enrichment

Operators like `kubemanager/`, `kubenameresolver/`, `socketenricher/`, and `localmanager/` map kernel-level data (mount namespace IDs, PIDs, network namespaces) to high-level concepts (Kubernetes pods, containers, DNS names).

## Key Conventions

### Go Code

- **Module:** `github.com/inspektor-gadget/inspektor-gadget`
- **License headers:** Every Go source file must start with the Apache 2.0 copyright header with the current year: `// Copyright 2026 The Inspektor Gadget authors`
- **Error wrapping:** Always use `fmt.Errorf("context: %w", err)` — enforced by `errorlint` linter.
- **Formatting:** `gofumpt` + `goimports` with local prefix `github.com/inspektor-gadget/inspektor-gadget` (imports grouped: stdlib, external, then local). All files must end with a newline.
- **Logging:** Use the `pkg/logger.Logger` interface (`Warnf`, `Debugf`, `Infof`, etc.), not `logrus` directly in library code.
- **Testing:** Uses `github.com/stretchr/testify` (`require` and `assert`). Table-driven tests are standard. Many tests require `sudo` (`go test -exec sudo`). Test-only helpers belong in `internal/testing/` or `pkg/testing/`, not in production packages.
- **Build tags:** Use `//go:build !withoutebpf` to guard code requiring eBPF. Use `//go:build linux` for Linux-only code.

### eBPF Code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inspektor-gadget/inspektor-gadget](https://github.com/inspektor-gadget/inspektor-gadget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
