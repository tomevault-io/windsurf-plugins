---
trigger: always_on
description: > **Performance Tip**: Use `Ctrl+F` to jump to sections using anchor links (e.g., `#building-and-running`)
---

# Panurus

> **Performance Tip**: Use `Ctrl+F` to jump to sections using anchor links (e.g., `#building-and-running`)

## 🚀 Quick Reference Commands

### Testing
- `make unit-tests` - Run unit tests
- `make unit-tests-race` - Unit tests with race detector
- `make integration-tests-fabtoken-fabric-t1` - Fabtoken integration tests
- `make integration-tests-dlog-fabric-t1 TEST_FILTER="T1"` - ZK integration tests with T1 filter

### Development & CI Preparation
- `make fmt` - Format code using gofmt
- `make lint` - Check code style
- `make lint-auto-fix` - Auto-fix linting issues (recommended pre-commit)
- `make install-tools` - Install development dependencies
- `make checks` - Run all pre-CI checks (license, fmt, vet, etc.)
- `make download-fabric` - Download Fabric binaries
- `make docker-images` - Prepare Docker images
- `make testing-docker-images` - Prepare test Docker images

### Maintenance
- `make clean` - Remove build artifacts
- `make clean-all-containers` - Remove Docker containers
- `make tidy` - Synchronize Go dependencies
- `go generate ./...` - Generate mocks

## 🔧 Development Workflow

### 1. Setup (One-time)
```bash
make install-tools
make download-fabric
export FAB_BINS=$PWD/../fabric/bin
make docker-images
make testing-docker-images
```

### 2. Daily Development
```bash
# Code quality
make lint-auto-fix
make checks

# Testing
make unit-tests          # Standard
make unit-tests-race     # With race detection
make integration-tests-fabtoken-fabric-t1  # Integration tests
```

### 3. Debugging
```bash
# Performance profiling
go test -cpuprofile=cpu.out ./...
go test -memprofile=mem.out ./...

# Focused testing
make integration-tests-dlog-fabric TEST_FILTER="T1"
```

## 🐛 Troubleshooting Quick Reference

- **Chaincode packaging failed**: Verify `FAB_BINS` is set correctly and points to valid Fabric binaries
- **Docker errors**: Run `make testing-docker-images`
- **Linting errors on commit**: Run `make lint-auto-fix`
- **Test timeouts**: Increase Docker resource allocation
- **Permission denied**: `chmod +x` on Fabric binaries in `$FAB_BINS`
- **Container conflicts**: `make clean-all-containers`
- **Go module issues**: `make tidy`
- **Mock generation failures**: `make install-tools` (ensures counterfeiter is installed)

## 🏗️ Architecture Overview

### Core Patterns
- **Driver Pattern**: Swappable token technologies via interfaces in `token/driver`
- **Service Pattern**: Encapsulated high-level logic in `token/services`
- **TTX Service**: Orchestrates token transaction lifecycle (Request → Assemble → Sign → Commit)

## 🧪 Testing Strategy

### Unit Tests
- Located alongside implementation code (`*_test.go`)
- Use testify for assertions (`assert` for values, `require` for error handling)
- Prefer table-driven tests for service logic
- Use context struct pattern to minimize mock boilerplate

### Integration Tests
- Located in `integration/` directory
- Utilize Network Orchestrator (NWO) for ephemeral Fabric networks
- Use `TEST_FILTER` environment variable with Ginkgo labels for focused testing
- Example: `TEST_FILTER="T1"` runs only tests with T1 label

### Fuzz Testing
- Add a `FuzzXxx` test (Go native fuzzing) wherever meaningful — any exported
  function that parses untrusted/attacker-controlled bytes (deserializers,
  wire-format decoders, signature/identity/token parsers) should get one,
  proactively when the entry point is added or touched, not only after a bug
  is found there.
- Seed the corpus with `f.Add(...)`: valid input, empty input, truncated/
  malformed input, and any known historical edge cases (e.g. a payload that
  previously triggered a panic).
- Verify locally before committing: `go test <pkg> -run='^$' -fuzz='^FuzzXxx$' -fuzztime=20s`
  with no panics, plus a plain `go test <pkg>` run to confirm the seed corpus
  passes as ordinary test cases.
- **Wire every new `FuzzXxx` target into `.github/workflows/nightly-fuzz.yml`**:
  add a `{name, pkg, func}` entry to the `fuzz` job's `strategy.matrix.include`
  list. A fuzz test that isn't in that matrix never actually runs under
  extended `-fuzztime` in CI — it only gets exercised by its seed corpus in
  the regular unit-test run.

### Mocking Best Practices
- Generate mocks with `counterfeiter` (`go generate ./...`)
- Use `disabled.Provider` for metrics to avoid nil panics
- Use `noop.NewTracerProvider()` for tracing
- Employ Context Struct + Setup Helper pattern (see `token/services/ttx` for example)

## 📝 Development Conventions

### Coding Standards
- **Error Handling**: Handle errors explicitly; avoid blank identifier for errors
- **Error Construction**: Never use `fmt.Errorf` (or `fmt` at all) to build or wrap errors. Always use `github.com/hyperledger-labs/fabric-smart-client/pkg/utils/errors` instead (`errors.New`, `errors.Errorf`, `errors.Wrap`, `errors.Wrapf`, `errors.WithMessage`, `errors.WithMessagef`, `errors.Join`, etc.). This applies to source code, tests, and code samples in `docs/`.
- **Interfaces**: Define small, focused interfaces on consumer side; favor composition
- **Concurrency**: Use goroutines and channels; avoid shared state; validate with race detector
- **Globals**: Avoid global variables for testability

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LFDT-Panurus/panurus](https://github.com/LFDT-Panurus/panurus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
