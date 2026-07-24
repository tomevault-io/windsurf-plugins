---
trigger: always_on
description: Tracee is a runtime security and observability tool that uses eBPF technology to tap into system activity and expose events ranging from factual system information to sophisticated security detections. It's a large, mature codebase (~500K+ lines) written primarily in Go and C (eBPF), targeting Linux systems with kernel 4.18+.
---

# Copilot Instructions for Tracee

## Repository Overview

Tracee is a runtime security and observability tool that uses eBPF technology to tap into system activity and expose events ranging from factual system information to sophisticated security detections. It's a large, mature codebase (~500K+ lines) written primarily in Go and C (eBPF), targeting Linux systems with kernel 4.18+.

### Key Technologies
- **Languages**: Go (1.24+), C (eBPF), Shell scripts
- **Build Tools**: Make, Clang 12+, Go toolchain
- **Frameworks**: eBPF/libbpf, gRPC, Kubernetes operators
- **Target Platforms**: Linux x86_64, aarch64

## Build Instructions & Environment Setup

### Essential Prerequisites
Always ensure these requirements before building:
- **Go 1.24+** (specifically required - earlier versions will fail)
- **Clang 12+** (minimum version enforced by Makefile)
- **pkg-config, build-essential, libelf-dev, libzstd-dev** (system packages)

### Core Build Commands

```bash
# ALWAYS start with environment check
make env                    # Shows all environment variables and tool versions

# Basic building (takes 1-2 minutes)
make clean                  # Clean all build artifacts first
make all                    # Builds tracee, signatures, evt, traceectl

# Individual components
make bpf                    # eBPF object only
make tracee                 # Main unified binary
make signatures             # Go plugin signatures
make tracee-bench           # Benchmarking tool
```

### Testing Commands

```bash
# Unit tests (takes ~6-8 seconds)
make test-unit              # Fast, comprehensive unit tests

# Integration tests (requires root privileges)
sudo make test-integration  # Some tests skip without root

# Types module testing
make test-types             # Separate Go module testing

# Common module testing
make test-common            # Common module testing
```

### Validation & Linting

**IMPORTANT**: Some validation commands require additional tools not installed by default:

```bash
# Code formatting (NEEDS goimports-reviser, clang-format-19)
make check-fmt              # Will fail without proper tools
make fix-fmt                # Auto-fix formatting issues

# Other validation
make check-vet              # Go vet checks
make check-staticcheck      # Static analysis (needs staticcheck tool)
make check-err              # Error checking (needs errcheck tool)
make check-vulncheck        # Vulnerability checking (needs govulncheck tool)
make check-pr               # Full PR validation suite
```

**Tool Installation**: If you get "missing required tool" errors, install via:
- `goimports-reviser` (v3.12.6): `go install github.com/incu6us/goimports-reviser/v3@fa5587e51ba33c58734984cb41370a5b2582d5b7`
- `staticcheck` (2025.1): `go install honnef.co/go/tools/cmd/staticcheck@5af2e5fc3b08ba46027eb48ebddeba34dc0bd02c`
- `errcheck` (v1.9.0): `go install github.com/kisielk/errcheck@11c27a7ce69d583465d80d808817d22d6653ee34`
- `govulncheck` (v1.1.4): `go install golang.org/x/vuln/cmd/govulncheck@d1f380186385b4f64e00313f31743df8e4b89a77`
- `clang-format-19`: Use system package manager or download binary

### Build Flags & Options

```bash
STATIC=1 make tracee        # Static binary linking
DEBUG=1 make tracee         # Debug symbols included
METRICS=1 make tracee       # Enable BPF metrics
BTFHUB=1 STATIC=1 make      # Embed BTF for kernel compatibility
```

### Common Build Issues & Solutions

1. **"missing required tool"**: Install the specific tool mentioned in error
2. **Go version errors**: Must use Go 1.24+, earlier versions fail validation
3. **Clang version errors**: Must use Clang 12+, checked by Makefile
4. **Integration test failures**: Many tests require root privileges
5. **libbpf build failures**: Check system has libelf-dev and build tools
6. **Permission errors in tests**: Integration tests need `sudo` for eBPF operations

## Project Architecture & Structure

### Core Directories

- **`cmd/`** - Command-line binaries
  - `tracee/` - Main unified Tracee binary
  - `evt/` - Event generation and testing tool
  - `traceectl/` - Control plane client tool (separate Go module)

- **`pkg/`** - Main Go packages and libraries
  - `ebpf/` - eBPF program management, event processing
  - `events/` - Event definitions, parsing, filtering
  - `signatures/` - Signature engine and framework
  - `policy/` - Policy management and enforcement
  - `containers/` - Container runtime integration
  - `proctree/` - Process tree tracking
  - `filters/` - Event filtering logic

- **`pkg/ebpf/c/`** - eBPF C source code
  - `tracee.bpf.c` - Main eBPF program
  - `common/` - Shared headers and utilities
  - `maps.h`, `types.h` - eBPF data structures

- **`signatures/golang/`** - Detection signatures (Go plugins)
  - Individual `.go` files implementing specific threat detections
  - `export.go` - Signature plugin exports

### Build Artifacts

All build outputs go to `./dist/` directory:
- `tracee` - Main unified binary
- `tracee.bpf.o` - Compiled eBPF object

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aquasecurity/tracee](https://github.com/aquasecurity/tracee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
