---
trigger: always_on
description: This is a Go event sourcing library **exploring** the Dynamic Consistency Boundary (DCB) pattern with PostgreSQL for production and SQLite for benchmark test data.
---

# Cursor Project Rules for go-crablet

# Go-Crablet Cursor Configuration

## Project Overview
This is a Go event sourcing library **exploring** the Dynamic Consistency Boundary (DCB) pattern with PostgreSQL for production and SQLite for benchmark test data.

**IMPORTANT: This is an exploration project, not a production-ready solution.**
- We are **learning and experimenting** with DCB concepts
- Performance claims should be **modest and factual**
- Emphasize **exploration** over **production readiness**
- Be **honest about limitations** and areas for improvement

## Comprehensive Testing Requirements

### Test Categories
- **Internal Tests**: Core DCB package tests (`pkg/dcb/`)
- **External Tests**: DCB test suite (`pkg/dcb/tests/`) and example applications (`internal/examples/`)
- **Benchmark Tests**: Performance validation (`internal/benchmarks/`)

### Mandatory Test Execution
**ALWAYS run both internal and external tests before considering any changes complete:**

```bash
# Run all internal tests (core DCB functionality)
go test ./pkg/dcb -v

# Run all external tests (DCB test suite and examples)
go test ./pkg/dcb/tests -v
go test ./internal/examples/... -v

# Run all tests comprehensively
go test ./... -v
```

### Test Validation Checklist
Before marking any task complete:
- [ ] Internal DCB tests pass (core functionality)
- [ ] External DCB test suite passes (advanced scenarios)
- [ ] External example tests pass (integration scenarios)
- [ ] All Ginkgo BDD tests pass (if applicable)
- [ ] No test failures or skipped tests
- [ ] Testcontainers integration working
- [ ] Database schema and functions created successfully

### Test Framework Requirements
- **Internal Tests**: Use Ginkgo v2 + Gomega for BDD testing
- **External Tests**: Support both standard Go tests and Ginkgo BDD
- **Database Tests**: Always use Testcontainers for isolated PostgreSQL instances
- **Coverage**: Maintain comprehensive test coverage for all public APIs

## Code Coverage System

### Coverage Calculation
The project uses a sophisticated coverage system with **gocovmerge** to combine coverage from multiple test suites:

1. **Internal Tests** (`pkg/dcb/`): ~15% coverage (31 Ginkgo specs)
2. **External Tests** (`pkg/dcb/tests/`): ~77% coverage (154 Ginkgo specs)
3. **Combined Coverage**: ~81% (using gocovmerge to merge coverage files)

### Coverage Scripts
- `scripts/generate-coverage.sh` - Generates comprehensive coverage using gocovmerge
- `scripts/update-coverage-badge.sh` - Updates README badge with coverage percentage
- GitHub Actions workflow (`.github/workflows/coverage.yml`) - Automated coverage reporting

### Coverage Workflow
```bash
# Generate comprehensive coverage
./scripts/generate-coverage.sh

# Update badge (optional)
./scripts/generate-coverage.sh update-badge
```

### Key Functions with Low Coverage (0%)
- `isConcurrencyError` (append.go:267) - Private helper, tested indirectly
- `NewEventStoreWithConfig` (constructors.go:40) - Used in docs/examples, tested through main constructor
- `NewCommandSimple` (constructors.go:216) - Simple wrapper, tested through NewCommand
- `IsTableStructureError` (errors.go:83) - Public API, may need tests if used
- `GetTableStructureError` (errors.go:120) - Public API, may need tests if used
- `AsConcurrencyError`, `AsResourceError`, `AsTableStructureError` (errors.go) - Simple aliases, tested through Get* functions
- `isInputEvent`, `isTag`, `isQuery`, `isQueryItem`, `isAppendCondition` (types.go) - Interface markers, no implementation to test

### Coverage Accuracy
The 80% coverage is legitimate because:
- **External tests are comprehensive** - they test main public APIs extensively
- **gocovmerge combines best coverage** - if a line is covered in either test suite, it counts
- **Core functionality is well-tested** - main EventStore, Append, Query, and Projection methods have good coverage
- **Low-coverage functions are edge cases** - many 0% functions are helper methods or alternative constructors

### Coverage Improvement Strategy
To improve coverage further:
1. **Add tests for 0% coverage functions** - especially error handling and alternative constructors
2. **Test edge cases** - concurrency errors, table structure errors, etc.
3. **Add integration tests** - for the config-based constructors
4. **Test helper functions** - the `is*` type checking functions

## SQLite Test Data System

### Key Directories
- `internal/benchmarks/setup/` - Dataset generation and SQLite caching
- `internal/benchmarks/tools/` - Dataset preparation tools
- `cache/` - SQLite database with pre-generated test datasets
- `internal/benchmarks/benchmarks/` - Go benchmark tests

### SQLite Test Data Workflow

1. **Generate Test Datasets**:
   ```bash
   cd internal/benchmarks/tools
   go run prepare_datasets_main.go
   ```
   This creates SQLite cache with "tiny" and "small" datasets.

2. **Run Go Benchmarks**:
   ```bash
   make benchmark-go
   # or
   cd internal/benchmarks/benchmarks
   go test -bench=. -benchmem -benchtime=2s -timeout=5m .
   ```

3. **Clear Cache** (if needed):
   ```bash
   rm -rf cache/
   ```

### Dataset Sizes
- **"tiny"**: 5 courses, 10 students, 20 enrollments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rodolfodpk/go-crablet](https://github.com/rodolfodpk/go-crablet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
