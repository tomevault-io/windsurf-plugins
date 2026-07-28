---
trigger: always_on
description: **minio-rs** is a MinIO Rust SDK for Amazon S3 compatible cloud storage. It provides a strongly-typed, async-first interface to MinIO and S3-compatible object storage APIs using a request builder pattern with full async/await support via tokio.
---

# Copilot Instructions for minio-rs

## Repository Overview

**minio-rs** is a MinIO Rust SDK for Amazon S3 compatible cloud storage. It provides a strongly-typed, async-first interface to MinIO and S3-compatible object storage APIs using a request builder pattern with full async/await support via tokio.

- **Language**: Rust (edition 2024)
- **Rust Version**: 1.88.0 (specified in `rust-toolchain.toml`)
- **Project Type**: Library crate with examples, integration tests, and benchmarks
- **Repository Size**: ~160 Rust source files, ~273 total files
- **License**: Apache-2.0

## Build Commands and Validation

### Prerequisites
- Rust toolchain 1.88.0 with clippy and rustfmt components (automatically installed via `rust-toolchain.toml`)
- No additional system dependencies required for basic builds

### Essential Commands (in order of typical workflow)

**ALWAYS run these commands in sequence before submitting changes:**

1. **Format Check** (~1 second):
   ```bash
   cargo fmt --all -- --check
   ```
   - Must pass with no output
   - Auto-fix: `cargo fmt --all`

2. **Clippy Linting** (~45-70 seconds from clean, instant if cached):
   ```bash
   cargo clippy --all-targets --all-features --workspace -- -D warnings
   ```
   - Fails on any warnings
   - This is the primary lint check used in CI

3. **Build** (~90 seconds for full build from clean, ~45 seconds for basic build, ~20 seconds incremental):
   ```bash
   cargo build --bins --examples --tests --benches --verbose
   ```
   - Builds all targets including examples, tests, and benchmarks
   - Basic library build: `cargo build` (~20-45 seconds)
   - Clean build: `cargo clean` first (removes 6-7 GB in `target/`)

4. **Unit Tests** (<1 second):
   ```bash
   cargo test --lib
   ```
   - Runs only library unit tests (3 tests)
   - Does NOT require MinIO server
   - Safe to run in any environment

5. **Integration Tests** (require MinIO server setup):
   ```bash
   # IMPORTANT: Integration tests require a running MinIO server
   ./tests/start-server.sh
   export SERVER_ENDPOINT=localhost:9000
   export ACCESS_KEY=minioadmin
   export SECRET_KEY=minioadmin
   export ENABLE_HTTPS=1
   export MINIO_SSL_CERT_FILE=./tests/public.crt
   
   # Run with multi-threaded runtime
   MINIO_TEST_TOKIO_RUNTIME_FLAVOR="multi_thread" cargo test -- --nocapture
   
   # OR run with current-thread runtime
   MINIO_TEST_TOKIO_RUNTIME_FLAVOR="current_thread" cargo test -- --nocapture
   ```
   - WITHOUT these environment variables, integration tests will FAIL
   - The server setup script downloads and starts MinIO locally
   - Tests are located in `tests/` directory (30+ test files)
   - Use `--test <test_name>` to run a specific test file

6. **Documentation** (~8-10 seconds):
   ```bash
   cargo doc --no-deps
   ```
   - Generates documentation in `target/doc/`
   - May show ~13 warnings about unresolved links (existing issue)
   - Full doc build: `cargo doc --all` (includes dependencies)

7. **Quick Check** (~29 seconds from clean):
   ```bash
   cargo check
   ```
   - Faster than build, useful for quick syntax validation

### Feature Flags
- **Default features**: `default-tls`, `default-crypto`
- **Crypto backends**: 
  - `default-crypto` (sha2 + hmac) - default
  - `ring` - alternative crypto backend
- **TLS backends**: 
  - `default-tls` - default
  - `native-tls` 
  - `rustls-tls`
- **Special**: `localhost` feature for local testing

⚠️ **IMPORTANT**: Building with `--no-default-features` will FAIL. Always use at least one crypto backend and one TLS backend:
```bash
cargo build --no-default-features --features "ring,default-tls"  # Works
cargo build --no-default-features  # FAILS
```

### Benchmarks
```bash
cargo bench --bench s3-api --no-run  # Build only (~93 seconds)
cargo bench --bench s3-api           # Run benchmarks (requires MinIO server)
```
- Results stored in `target/criterion/`
- See `benches/README.md` for details

### Examples
```bash
cargo run --example file_uploader
cargo run --example file_downloader
cargo run --example object_prompt
```
- Examples require network access to MinIO (defaults to localhost:9000)
- Will fail if network is unavailable
- Located in `examples/` directory (10 examples)

## Project Architecture

### Directory Structure
```
minio-rs/
├── src/                    # Main library source
│   ├── lib.rs             # Root library file, exports s3 module
│   └── s3/                # S3 API implementation
│       ├── client.rs      # MinioClient implementation (~27K lines)
│       ├── builders.rs    # Request builder exports
│       ├── builders/      # Individual request builders (40+ files)
│       ├── response.rs    # Response type exports
│       ├── response/      # Response types (40+ files)
│       ├── types.rs       # Core types (S3Api trait, etc.)
│       ├── error.rs       # Error types
│       ├── utils.rs       # Utility functions
│       ├── signer.rs      # AWS signature v4
│       ├── creds.rs       # Credentials providers
│       ├── http.rs        # HTTP utilities
│       └── ...            # Other modules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minio/minio-rs](https://github.com/minio/minio-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
