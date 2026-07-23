---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
with code in this repository.

## Project Overview

Delve is a debugger for the Go programming language. This is a complex,
multi-layered system that requires understanding of debugging internals,
DWARF format, OS-specific process control, and Go runtime internals.

## Build and Test Commands

### Building

```bash
make build          # Build dlv binary
make install        # Install dlv to system
make uninstall      # Remove dlv from system
```

### Testing

```bash
make test                       # Run all tests with vetting
make vet                        # Run Go vet with architecture tags
go test -run TestName ./pkg/... # Run specific test by name
go test ./pkg/proc              # Run all pkg/proc tests
go test ./service/test          # Run all integration tests
```

### eBPF Backend Development

```bash
make build-ebpf-image   # Build Docker image for eBPF compilation
make build-ebpf-object  # Compile eBPF C code to object files
```

The eBPF backend uses Docker to compile C code
(`pkg/proc/internal/ebpf/bpf/trace.bpf.c`) in a controlled environment,
producing architecture-specific `.o` files.

The builder image (`pkg/proc/internal/ebpf/build/ebpf-Dockerfile`) is based
on Ubuntu 26.04 with **clang-22**. The eBPF C code uses a per-CPU scratch
buffer with `bpf_ringbuf_output` for variable-length events, which requires
clang 14+ for correct bounded-offset codegen on `PTR_TO_MAP_VALUE`.

#### eBPF Testing

eBPF tests (TestTraceEBPF*) require elevated capabilities (CAP_BPF,
CAP_PERFMON, CAP_SYS_RESOURCE) and must be run with sudo:

```bash
# Run specific eBPF test
sudo go test -v -run TestTraceEBPF3 -count 1 ./cmd/dlv

# Run all eBPF tests
sudo go test -v -run TestTraceEBPF -count 1 ./cmd/dlv

# Run eBPF tests in Docker (useful when host lacks capabilities or for CI)
docker run --privileged -v "$(pwd)":/delve -w /delve \
  -e GOFLAGS="-buildvcs=false" golang:1.24-bookworm \
  go test -v -run TestTraceEBPF -count 1 ./cmd/dlv

# Suppress debug output
sudo go test -v -run TestTraceEBPF3 -count 1 ./cmd/dlv 2>&1 | \
  grep -v "^DEBUG"
```

### Running Delve

```bash
dlv debug           # Compile and debug current package
dlv test            # Compile and debug tests
dlv attach <pid>    # Attach to running process
dlv exec <binary>   # Debug pre-compiled binary
dlv dap             # Start Debug Adapter Protocol server
```

## Test-Driven Development (MANDATORY)

**CRITICAL**: All code changes MUST follow test-driven development (TDD).
This is not optional.

### Red-Green-Refactor Cycle

1. **RED** - Write a failing test first (must fail for the right reason)
2. **GREEN** - Write minimum code to pass the test
3. **REFACTOR** - Clean up while keeping tests green
4. **VERIFY** - Run full test suite before committing

### Mandatory TDD Rules

1. **NEVER write implementation code without a failing test first**
2. **NEVER commit code without tests**
3. **NEVER skip tests or mark them as skipped to make CI pass**
4. **NEVER disable existing tests** - fix them or fix the code
5. Each test should verify ONE specific behavior
6. Test names should describe the scenario being tested

### Example Workflow

```bash
# RED - Write failing test
go test -run TestEvaluateComplexType ./pkg/proc
# Output: FAIL - undefined: evaluateComplexType (EXPECTED)

# GREEN - Implement minimum code
go test -run TestEvaluateComplexType ./pkg/proc
# Output: PASS

# REFACTOR - Clean up
go test -run TestEvaluateComplexType ./pkg/proc
# Output: PASS (must stay green)

# VERIFY - Full suite
make test
```

### Where to Write Tests

- **Unit tests**: Co-locate with code (e.g., `pkg/proc/variables_test.go`
  for `variables.go`)
- **Integration tests**: `service/test/` for end-to-end scenarios
- **Platform-specific tests**: Use build tags (e.g., `//go:build linux`)
- **Backend-specific tests**: Test each backend separately
- **Test fixtures**: Source code in `_fixtures/` (compiled during tests,
  not pre-compiled binaries)
- **Finding fixtures directory**: Use `protest.FindFixturesDir()` from
  `github.com/go-delve/delve/pkg/proc/test` instead of writing custom
  fixture directory lookup code

### Test Quality Standards

- Test behavior, not implementation
- One assertion per test when possible (easier diagnosis)
- Clear naming: `Test<What>_<Scenario>_<ExpectedResult>`
- Use existing test utilities in `*_test.go` files
- Tests must be deterministic (no random values or race conditions)

### When TDD Seems Difficult

Difficulty writing tests first indicates: (1) function doing too much,
(2) tightly coupled dependencies, or (3) unclear requirements. **DO NOT
skip TDD** - the difficulty signals a design problem.

## Architecture Overview

Delve uses a **layered architecture** with clear separation of concerns:

```
CLI (cmd/dlv) → Cobra commands
    ↓
Service Layer (service/) → RPC2, DAP protocol implementations
    ↓
Debugger (service/debugger) → High-level debugging operations
    ↓
Process Abstraction (pkg/proc) → TargetGroup, Target, ProcessInternal
    ↓
Backend Implementations:
  - pkg/proc/native/        → Direct OS debugging (ptrace, Windows APIs)
  - pkg/proc/gdbserial/     → GDB remote protocol client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-delve/delve](https://github.com/go-delve/delve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
