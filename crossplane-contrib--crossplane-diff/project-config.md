---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Testing

```bash
# Build the binary for your platform
earthly +build

# Run unit and integration tests (fast, ~7s)
# Direct go test - fastest, immediate output
cd cmd/diff
go test ./...

# Via Earthly (ensures consistent environment, caches dependencies)
earthly +go-test

# Run a specific test
go test ./cmd/diff/diffprocessor -run TestCachedFunctionProvider -v

# Run a single test file
go test ./cmd/diff/diffprocessor/diff_processor_test.go -v

# Check test coverage
go test -cover ./cmd/diff/diffprocessor/... -coverprofile=/tmp/coverage.out
go tool cover -func=/tmp/coverage.out

# Pre-PR checks: linting, tests, generation (requires long timeout, can take several minutes)
earthly -P +reviewable

# Fetch Crossplane cluster CRDs (required after Crossplane API changes or for integration tests)
earthly +fetch-crossplane-cluster --CROSSPLANE_IMAGE_TAG=main

# Tidy go modules
earthly +generate
```

**Earthly Output Notes:**
- By default, Earthly buffers stdout and stderr separately, which can cause interleaved output
- Use `2>&1` to merge streams for chronological output: `earthly +go-test 2>&1 | tee output.log`

### End-to-End Tests

E2E tests run against real kind clusters with Crossplane installed. They can take several minutes to complete.

```bash
# Full E2E matrix against multiple Crossplane versions (slow, runs serially)
earthly -P +e2e-matrix

# Single E2E test against specific Crossplane version
earthly +e2e --CROSSPLANE_IMAGE_TAG=main

# Run specific E2E test with verbose logging
earthly -P +e2e --FLAGS="-v=4 -test.run ^TestCompositionDiff"

# Debug E2E: stop on first failure and preserve kind cluster
earthly -i -P +e2e --FLAGS="-test.failfast -fail-fast -destroy-kind-cluster=false"

# Run E2E tests directly (without Earthly wrapper for easier debugging)
go test -c -o e2e ./test/e2e
./e2e -v=4 -test.v -test.failfast -destroy-kind-cluster=false -test.run ^TestSpecificTest
```

**IMPORTANT**: Never interrupt running tests to try a simpler approach. E2E tests take a long time but that's expected. Killing them wastes the effort up to that point.

**Test Output Management**: Tests can take several minutes to run. Always save test output to an intermediate file before processing:
```bash
# Good: Save to file first, then query
earthly -P +e2e --test_name=TestFoo 2>&1 | tee /tmp/test-output.log
grep -A50 "FAIL" /tmp/test-output.log

# Bad: Pipe directly to grep (wastes test run if you need different info)
earthly -P +e2e --test_name=TestFoo 2>&1 | grep "FAIL"
```

**Debugging Test Failures**: When E2E tests fail, check `_output/tests/e2e-tests.xml` for complete, un-truncated failure output:
```bash
# Extract specific test failure from XML (includes full output)
grep -A 100 "TestDiffCompositionWithGetComposedResource" _output/tests/e2e-tests.xml

# Note: Earthly output quirks
# - First emits the failure
# - Then emits the log of the run
# - Finally repeats the failure with "*failed*" prepended to each line
```

### Running the CLI

```bash
# Build and run locally
earthly +build
./_output/bin/darwin_arm64/crossplane-diff xr test-xr.yaml

# XR diff - compare XR against cluster state
crossplane-diff xr my-xr.yaml
crossplane-diff xr my-xr.yaml --compact --no-color

# Composition diff - see impact of composition changes on existing XRs
crossplane-diff comp updated-composition.yaml
crossplane-diff comp updated-composition.yaml -n production --include-manual
```

## Architecture

### High-Level Structure

The codebase follows a clean layered architecture with dependency injection and separation of concerns:

```
cmd/diff/
├── main.go                    # CLI entry point (kong-based argument parsing)
├── xr.go                      # XR diff command implementation
├── comp.go                    # Composition diff command implementation
├── client/                    # Kubernetes and Crossplane API clients
│   ├── crossplane/           # Crossplane-specific clients (Compositions, XRDs, Functions, etc.)
│   ├── kubernetes/           # Generic Kubernetes clients (CRDs, dynamic client)
│   └── core/                 # Core client interfaces
├── diffprocessor/            # Core diff logic (domain layer)
│   ├── diff_processor.go     # Main diff orchestration for XRs
│   ├── comp_processor.go     # Composition diff orchestration
│   ├── diff_calculator.go    # Calculates diffs between resources
│   ├── resource_manager.go   # Fetches current cluster state
│   ├── schema_validator.go   # Validates resources against CRD schemas
│   ├── requirements_provider.go  # Resolves composition requirements
│   ├── function_provider.go  # Provides functions for composition pipeline
│   └── processor_config.go   # Configuration and dependency injection
├── renderer/                 # Crossplane render pipeline wrapper
├── testutils/                # Test helpers and mock builders
└── types/                    # Shared types and interfaces
```

### Key Architectural Patterns

1. **Dependency Injection via Factory Pattern**
   - Processors are configured using functional options pattern (`ProcessorOption`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crossplane-contrib/crossplane-diff](https://github.com/crossplane-contrib/crossplane-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
