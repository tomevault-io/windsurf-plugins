---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

rustbridge is a framework for developing Rust shared libraries callable from other languages (Java, C#, Python, etc.). It uses C ABI under the hood but abstracts the complexity, providing OSGI-like lifecycle, mandatory async (Tokio), logging callbacks, and JSON-based data transport with optional binary transport for performance-critical paths.

## Quick Reference

```bash
# Pre-commit validation (run before committing)
./scripts/pre-commit.sh              # Full validation (Linux/macOS)
./scripts/pre-commit.sh --fast       # Skip clippy and integration tests
./scripts/pre-commit.sh --smart      # Only test changed components
scripts\pre-commit.bat               # Windows (full validation)
scripts\pre-commit.bat --fast        # Windows (skip clippy/integration)

# Common development commands
cargo fmt --all                                                    # Format code
cargo clippy --workspace --examples --tests -- -D warnings         # Lint (must pass)
cargo test --workspace                                             # Test all crates
cargo test -p rustbridge-ffi                                       # Test specific crate
cargo test lifecycle___installed                                   # Run tests matching pattern
cargo bench -p rustbridge-transport -- small_roundtrip             # Run specific benchmark

# Bundle operations
rustbridge pack                                                    # Auto-detect and bundle from current dir
rustbridge pack --no-sign                                          # Bundle without signing
rustbridge promote plugin-dev.rbp --key secret.key -o plugin.rbp  # Slim dev bundle to signed release
rustbridge bundle create --name my-plugin --version 1.0.0 \
  --lib linux-x86_64:target/release/libmyplugin.so --output plugin.rbp  # Manual bundle creation

# Build native library (required before Java/C#/Python tests)
cargo build --release -p hello-plugin                             # Example plugin
# Output: target/release/libhello_plugin.so (Linux), .dylib (macOS), .dll (Windows)

# Java/Kotlin (from rustbridge-java/)
./gradlew build && ./gradlew test                                 # Linux/macOS
./gradlew test --tests "*PluginTest*"                             # Run tests matching pattern
gradlew.bat build && gradlew.bat test                             # Windows

# Python (from rustbridge-python/)
source .venv/bin/activate && python -m pytest tests/ -v
python -m pytest tests/test_log_level.py::test_log_level___debug___has_correct_value -v  # Single test
```

## Common Workflows

1. **Making code changes**: Edit → `cargo fmt --all` → `cargo clippy --workspace --examples --tests -- -D warnings` → `cargo test -p <changed-crate>`
2. **Before committing**: Run `./scripts/pre-commit.sh --smart` (tests only changed components)
3. **Full validation**: Run `./scripts/pre-commit.sh` (required before PRs)
4. **Cross-language changes**: If modifying FFI code in Rust, rebuild native lib (`cargo build --release`), then run Java/C#/Python tests

## Version Requirements

| Component | Minimum Version |
|-----------|----------------|
| Rust | 1.90.0 (Edition 2024) |
| Java | 21+ (22+ recommended) |
| .NET | 8.0+ |
| Python | 3.10+ |
| Go | 1.21+ |
| Erlang/OTP | 27+ |

**Note**: Java 21 requires `--enable-preview` flag in addition to `--enable-native-access=ALL-UNNAMED`. Java 22+ only needs `--enable-native-access=ALL-UNNAMED`.

Use `cargo msrv verify` when adding Rust dependencies.

## Versioning

Each language ecosystem is versioned independently. There is no lock-step version across ecosystems.

| Ecosystem | Registry | Version source |
|-----------|----------|----------------|
| Rust (12 crates) | crates.io | `Cargo.toml` workspace version (all crates share one version) |
| Java/Kotlin | Maven Central | `rustbridge-java/build.gradle.kts` `allprojects { version }` |
| C# | NuGet | `RustBridge.Core.csproj` and `RustBridge.Native.csproj` `<Version>` |
| Python | PyPI | `rustbridge-python/pyproject.toml` `version` |
| Erlang | hex.pm | `rustbridge-erlang/src/rustbridge.app.src` `{vsn, ...}` |
| Go | Go modules | Git tags (no version in `go.mod`) |

**Key rules:**
- All 12 Rust workspace crates are always versioned in lock-step via `[workspace.package] version` in the root `Cargo.toml`.
- Bump only the ecosystems that actually changed. Don't publish empty releases.
- When updating CLI templates (`crates/rustbridge-cli/templates/`), use the latest published version for each ecosystem's package references.
- CHANGELOG entries use ecosystem prefixes (`**C#**:`, `**Python**:`, `**Rust**:`, etc.) and release headers list which packages were published.

See [docs/VERSIONING.md](./docs/VERSIONING.md) for the full policy.

## Architecture Overview

```
Host Language → FFI Boundary → Async Runtime → Plugin Implementation → Response → FFI → Host
```

**Layered crate structure:**
- **Core** (`rustbridge-core`, `rustbridge-transport`): Traits, types, serialization
- **Runtime** (`rustbridge-runtime`, `rustbridge-logging`): Tokio integration, tracing callbacks
- **FFI** (`rustbridge-ffi`): C ABI exports, buffer management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jrobhoward/rustbridge](https://github.com/jrobhoward/rustbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
