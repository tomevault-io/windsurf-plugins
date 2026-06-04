---
trigger: always_on
description: Development commands and quality checks for mocks project
---


# Development Workflow for Mocks Project

This rule defines essential development commands, quality checks, and workflow patterns for the mocks CLI tool.

## Build and Run Commands

### Basic Build Operations
```bash
# Debug build (fast compilation, includes debug info)
cargo build

# Release build (optimized, smaller binary)
cargo build --release

# Check compilation without building binary
cargo check

# Build documentation
cargo doc --open
```

### Running the Application
```bash
# Run with default settings
cargo run -- run storage.json

# Run with custom host and port
cargo run -- run -H 127.0.0.1 -p 8080 storage.json

# Run without file modifications (read-only mode)
cargo run -- run --no-overwrite storage.json

# Initialize new storage file
cargo run -- init storage.json

# Initialize empty storage file
cargo run -- init --empty storage.json

# View help information
cargo run -- --help
cargo run -- run --help
cargo run -- init --help
```

### Environment Variables
```bash
# Disable colored output
NO_COLOR=1 cargo run -- run storage.json

# Enable debug file overwrite tracking
MOCKS_DEBUG_OVERWRITTEN_FILE=1 cargo run -- run storage.json
```

## Testing Commands

### Unit and Integration Tests
```bash
# Run all tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Run specific test
cargo test test_parse_socket_addr

# Run tests in specific module
cargo test storage::tests

# Run tests with specific pattern
cargo test "test_*_operation"

# Run tests in parallel (default) or single-threaded
cargo test -- --test-threads=1
```

### End-to-End Testing
```bash
# Run E2E tests with runn
cd runn-e2e
runn run runbooks/test.yml --verbose

# Run specific E2E test
runn run runbooks/test_post.yml

# Run E2E tests with debug output
RUST_LOG=debug runn run runbooks/test.yml
```

### Coverage Generation
```bash
# Install coverage tool (one-time setup)
cargo install cargo-llvm-cov --locked

# Generate coverage report (HTML)
cargo llvm-cov --html

# Generate coverage report (terminal output)
cargo llvm-cov

# Coverage for specific test
cargo llvm-cov --test integration_tests

# Coverage with specific output format
cargo llvm-cov --lcov --output-path coverage.lcov
```

## Code Quality Commands

### Formatting
```bash
# Format all code
cargo fmt

# Check if code is formatted without making changes
cargo fmt -- --check

# Format specific file
cargo fmt src/main.rs

# Format with specific config
cargo fmt -- --config imports_granularity=Crate
```

### Linting with Clippy
```bash
# Run clippy with default settings
cargo clippy

# Run clippy with strict warnings (project standard)
cargo clippy -- -D warnings

# Run clippy on all targets
cargo clippy --all-targets -- -D warnings

# Run clippy with specific lint levels
cargo clippy -- -W clippy::pedantic -D warnings

# Fix clippy suggestions automatically (where possible)
cargo clippy --fix
```

### MSRV (Minimum Supported Rust Version) Checks
```bash
# Install MSRV checker (one-time setup)
cargo install cargo-msrv --locked

# Find minimum supported Rust version
cargo msrv find

# Verify current MSRV (1.78.0 for this project)
cargo msrv verify

# List Rust versions for testing
cargo msrv list
```

## Security and Dependency Management

### Security Auditing
```bash
# Install cargo-audit (one-time setup)
cargo install cargo-audit --locked

# Run security audit
cargo audit

# Audit with JSON output
cargo audit --json

# Fix security vulnerabilities (where possible)
cargo audit fix
```

### Dependency Management
```bash
# Update dependencies
cargo update

# Add new dependency
cargo add serde_json

# Add development dependency
cargo add --dev tempfile

# Remove dependency
cargo remove unused_crate

# Check for outdated dependencies
cargo outdated
```

## Performance and Optimization

### Benchmarking
```bash
# Run benchmarks (if any exist)
cargo bench

# Profile with perf (Linux)
cargo build --release
perf record --call-graph=dwarf target/release/mocks run storage.json
perf report

# Memory profiling with valgrind
cargo build --release
valgrind --tool=massif target/release/mocks run storage.json
```

### Binary Size Optimization
```bash
# Build with size optimization
RUSTFLAGS="-C target-cpu=native" cargo build --release

# Strip binary (reduce size)
strip target/release/mocks

# Check binary size
ls -lh target/release/mocks

# Analyze binary composition
cargo bloat --release
```

## Development Environment Setup

### Required Tools
```bash
# Install required tools for development
cargo install cargo-llvm-cov --locked
cargo install cargo-msrv --locked
cargo install cargo-audit --locked
cargo install cargo-outdated --locked
cargo install cargo-bloat --locked

# Install runn for E2E testing
go install github.com/k1LoW/runn/cmd/runn@latest
# or
brew install k1LoW/tap/runn
```

### Git Hooks Setup
```bash
# Create pre-commit hook for quality checks
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
set -e

echo "Running pre-commit checks..."

# Format check
cargo fmt -- --check

# Lint check
cargo clippy -- -D warnings

# Test check
cargo test

echo "All checks passed!"
EOF

chmod +x .git/hooks/pre-commit
```

## Continuous Integration Workflow

### Local CI Simulation
```bash
# Run the same checks as CI locally
#!/bin/bash
set -e


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mocks-rs/mocks](https://github.com/mocks-rs/mocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
