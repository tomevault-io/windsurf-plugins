---
trigger: always_on
description: This file provides guidance for AI agents (like Claude Code) working on the CCSDS 124.0-B-1 project.
---

# Instructions for AI Agents

This file provides guidance for AI agents (like Claude Code) working on the CCSDS 124.0-B-1 project.

## Project Overview

This repository provides multi-language implementations of the CCSDS 124.0-B-1 lossless compression algorithm designed for spacecraft housekeeping data. This is a **monorepo** with independent implementations in C, C++, Python, Go, Rust, and Java.

## Repository Structure

```
ccsds124/
├── implementations/
│   ├── c/          # C implementation (embedded/flight)
│   ├── cpp/        # C++ implementation (embedded/flight)
│   ├── python/     # Python implementation (MicroPython compatible)
│   ├── go/         # Go implementation
│   ├── rust/       # Rust implementation
│   └── java/       # Java implementation (enterprise/ground)
├── crossvalidation/                    # Shared cross-validation infrastructure
│   ├── file_list.csv                   # Canonical manifest (expected sizes + SHA-256)
│   ├── run_crossvalidation.sh          # Shared runner (parameterized for binary paths)
│   ├── c/                              # C harness (encoder + decoder + Dockerfile)
│   ├── sandbox/                        # ESA reference code harness
│   ├── cpp/                            # C++ harness (TODO: #93)
│   ├── go/                             # Go harness (TODO: #93)
│   ├── rust/                           # Rust harness (TODO: #93)
│   └── java/                           # Java harness (TODO: #93)
├── docs/           # Shared documentation
└── test-vectors/   # Shared test data
```

## Key Principles

1. **Independent Versioning**: Each language has its own version (e.g., `c/v1.0.0`, `python/v0.9.0`)
2. **Interoperability**: All implementations must be able to compress/decompress each other's output
3. **Shared Test Vectors**: Use common test data in `test-vectors/` for validation
4. **No Over-Engineering**: Keep implementations simple and focused on the task

## Versioning Rules

- Use **prefixed git tags**: `c/vX.Y.Z`, `cpp/vX.Y.Z`, `python/vX.Y.Z`, `go/vX.Y.Z`, `rust/vX.Y.Z`, `java/vX.Y.Z`
- Follow [Semantic Versioning](https://semver.org/)
- Update version in language-specific files:
  - C: `implementations/c/VERSION` and `implementations/c/include/ccsds124.h`
  - C++: `implementations/cpp/VERSION` and `implementations/cpp/include/ccsds124/config.hpp`
  - Python: `implementations/python/pyproject.toml` and `implementations/python/ccsds124/__init__.py`
  - Go: Git tag (Go uses tags directly)
  - Rust: `implementations/rust/Cargo.toml`
  - Java: `implementations/java/pom.xml` and `implementations/java/VERSION`

## Commit Conventions

Use [Conventional Commits](https://www.conventionalcommits.org/) with language prefix.

**IMPORTANT**: When working on a GitHub issue, include the issue number in the commit message title:

```
c: add compression function (#5)
python: fix decompression bug (#12)
go: update benchmarks (#8)
docs: clarify algorithm overview (#3)
chore: update build system (#1)
```

For commits without an associated issue:
```
c: add compression function
python: fix decompression bug
```

## When Working on Implementations

### C Implementation
- Target embedded systems and spacecraft processors
- Minimize dynamic memory allocation
- Use fixed-size integer types (`uint8_t`, etc.)
- Provide clear error codes
- Test with Makefile: `cd implementations/c && make test`
- Cross-validate: `docker-compose run --rm c-crossvalidation` (results saved to `implementations/c/build/crossvalidation-results/crossvalidation-results.txt`)

### C++ Implementation
- Target embedded systems with C++17 support
- Header-only templates for compile-time size optimization
- Zero dynamic allocation (embedded-friendly)
- Works with `-fno-exceptions -fno-rtti`
- Uses Catch2 for testing
- Test with Makefile: `cd implementations/cpp && make test`
- Format code: `clang-format -i`

### Python Implementation
- Include type hints for all public APIs
- Follow PEP 8 style guidelines
- Use pytest for testing
- Maintain compatibility with Python 3.9+ (per `pyproject.toml` `requires-python`)
- Install dev dependencies: `pip install -e ".[dev]"`

### Go Implementation
- Follow Go idioms and conventions
- Return errors, don't panic
- Include benchmarks for performance-critical code
- Run tests: `go test ./...`
- Format code: `go fmt ./...`

### Rust Implementation
- Follow Rust idioms and ownership patterns
- Use `Result<T, E>` for error handling
- Include benchmarks with Criterion
- Run tests: `cargo test`
- Format code: `cargo fmt`
- Lint: `cargo clippy`

### Java Implementation
- Target enterprise/ground systems (JDK 11+)
- Zero runtime dependencies (test-only dependencies allowed)
- Use Maven for builds
- Follow Google Java Style Guide
- Run tests: `mvn test`
- Format code: `mvn spotless:apply`
- Lint: `mvn checkstyle:check` and `mvn spotbugs:check`

## Testing Requirements

1. **Unit Tests**: Test individual functions
2. **Round-trip Tests**: Compress then decompress, verify original data
3. **Test Vectors**: Validate against shared test vectors in `test-vectors/`
4. **Cross-Implementation**: Verify interoperability between languages

## Documentation

- Keep language-specific docs in each implementation's README

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanagraspace/ccsds124](https://github.com/tanagraspace/ccsds124) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
