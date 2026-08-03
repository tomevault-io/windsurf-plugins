---
trigger: always_on
description: This guide provides comprehensive instructions for AI agents working on the jet1090/rs1090 project.
---

# Agent development guide

This guide provides comprehensive instructions for AI agents working on the jet1090/rs1090 project.

## Project overview

- Real-time enriched trajectory data serving
- Cross-platform export formats (JSON, gRPC, Arrow)
- Inspired by [pyModeS](https://github.com/junzis/pyModeS) library design
- Uses [deku](https://github.com/sharksforarms/deku) for declarative binary data decoding

## Project structure

```text
jet1090/
├── crates/
│   ├── rs1090/          # Core decoding library (Mode S, ADS-B, FLARM)
│   ├── jet1090/         # Live decoding application with TUI and web server
│   ├── decode1090/      # Companion CLI decoding tool
│   └── rs1090-wasm/     # WebAssembly bindings for browser usage
├── python/              # Python bindings (PyO3/maturin)
├── docs/                # MkDocs documentation (deployed to mode-s.org/jet1090)
├── samples/             # Real flight trajectory data for testing (private)
├── references/          # ADS-B/Mode S specification PDFs (ICAO standards, private)
└── container/           # Docker/Podman container definitions
```

### Crate responsibilities

- **rs1090**: Core library with decoding logic, CPR algorithms, data sources (RTL-SDR, SeRo, SSH, Beast)
- **jet1090**: Full-featured application with TUI, web server, snapshot management, deduplication
- **decode1090**: Lightweight CLI tool for batch message decoding
- **rs1090-wasm**: Browser-compatible WebAssembly bindings
- **python/**: Python bindings exposing `decode()` and `flarm()` functions

## Setup and build

### Initial build

```sh
# Development build (thin LTO, faster: ~47s incremental, keeps symbols)
cargo build --release --all-features

# Distribution build (full LTO, optimal binary: ~94s incremental, stripped)
cargo build --profile dist --all-features
```

**Build profiles:**

- `--release`: Thin LTO for fast development iteration (~15-16 MB with symbols, 47s incremental)
- `--profile dist`: Full LTO for production releases (12 MB stripped, 94s incremental)
  - Used automatically by `cargo dist` for releases

### Building specific components

```sh
# Core library only
cargo build -p rs1090 --release

# jet1090 application (development)
cargo build -p jet1090 --release

# jet1090 application (distribution)
cargo build -p jet1090 --profile dist

# Python bindings (requires uv)
cd python
uv sync --all-extras --dev
uv run maturin develop

# WebAssembly bindings
cd crates/rs1090-wasm
just pkg                    # Build bundled, web, and nodejs packages
# or, for a single target:
wasm-pack build --target web
```

### Nix platform

```sh
nix develop              # Enter development environment
nix build                # Build jet1090 (default package)
nix run                  # Run jet1090 directly
nix profile install      # Install to PATH
```

## Testing

### Rust tests

```sh
# Run all tests (workspace-wide)
cargo test --workspace --all-features --all-targets

# Run tests for specific crate
cargo test -p rs1090 --all-features

# Run specific test
cargo test test_name -- --nocapture

# Run with Nix
nix run .#checks.test-check  # Uses cargo-nextest
```

### Benchmarks

```sh
# Run Rust benchmarks
cargo bench

# Run specific benchmark
cargo bench --bench long_flight

# Python benchmarks
cd python/examples
uv run benchmark.py
```

### Python tests

```sh
cd python
uv run pytest                # Run all tests
uv run pytest tests/test_adsb.py  # Specific test file
uv run pytest -v             # Verbose output
```

### WebAssembly tests

```sh
cd crates/rs1090-wasm
just pkg
cd tests
npm install
npm test
```

The release workflow publishes `rs1090-wasm` to npm from `.github/workflows/wasm.yml` using npm Trusted Publishing/OIDC (`NPM_CONFIG_PROVENANCE=true`). If npm publishing fails with a 404/permission error on a tag, check the npm package trusted publisher settings:

- Publisher: GitHub Actions
- Repository: `xoolive/jet1090`
- Workflow filename: `wasm.yml`
- Environment: empty unless the workflow is changed to use one

## Code quality and style

### Rust

**Linting:**

```sh
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

**Formatting:**

```sh
cargo fmt --all              # Format all code
cargo fmt --all --check      # Check without modifying
```

**Documentation:**

```sh
cargo doc --all-features --no-deps        # Build docs
cargo doc --all-features --no-deps --open # Build and open in browser

# Check for documentation issues
RUSTDOCFLAGS="-D rustdoc::all -A rustdoc::private-doc-tests" cargo doc --all-features --no-deps
```

### Python

```sh
cd python
uv run ruff check            # Linting
uv run ruff format           # Formatting
uv run ruff format --check   # Check formatting without modifying

# Type checking
uv run ty check rs1090
```

### Markdown

- Use `prettier` for formatting documentation and markdown files
- Follow CommonMark specification

### Code conventions

**Rust:**

- Use descriptive variable names (e.g., `icao24`, `latitude_cpr`, `groundspeed`)
- Prefer declarative deku attributes for binary decoding
- Document public APIs with `///` doc comments
- Use `tracing` for logging, not `println!`
- Handle errors with `Result<T, E>`, avoid unwrap in library code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xoolive/jet1090](https://github.com/xoolive/jet1090) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
