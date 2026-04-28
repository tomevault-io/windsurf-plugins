---
trigger: always_on
description: Audiobook Forge is a blazing-fast CLI tool for converting audiobook directories to M4B format with chapters and metadata. This is a complete Rust rewrite of the original Python version, delivering 3-4x faster performance with true multi-core utilization.
---

# AGENTS.md - Audiobook Forge (Rust)

## Project Overview

Audiobook Forge is a blazing-fast CLI tool for converting audiobook directories to M4B format with chapters and metadata. This is a complete Rust rewrite of the original Python version, delivering 3-4x faster performance with true multi-core utilization.

**Key Capabilities**:
- Parallel processing of multiple audiobooks
- Smart quality detection and preservation
- Chapter generation from multiple sources (files, CUE sheets, auto-detection)
- Metadata extraction and enhancement (ID3, M4A tags)
- Cover art detection and embedding
- Batch operations on entire libraries
- Error recovery with automatic retry
- Hardware acceleration support (Apple Silicon aac_at encoder)

**Project Status**: All 6 development phases complete. See `docs/development/phases/` for detailed implementation history.

---

## Build & Test Commands

### Building
```bash
# Development build
cargo build

# Release build (optimized)
cargo build --release

# Binary location
./target/debug/audiobook-forge    # debug
./target/release/audiobook-forge  # release
```

### Testing
```bash
# Run all tests
cargo test

# Run library tests only
cargo test --lib

# Run with output
cargo test -- --nocapture

# Run specific test
cargo test test_name
```

### Running
```bash
# Check dependencies
cargo run -- check

# Process audiobooks
cargo run -- build --root /path/to/audiobooks --parallel 4

# Organize library
cargo run -- organize --root /path/to/audiobooks

# Config management
cargo run -- config init
cargo run -- config show
cargo run -- config validate
```

---

## Architecture

### Module Structure

```
src/
├── main.rs              # CLI entry point
├── lib.rs               # Library root
├── models/              # Core data structures
│   ├── book.rs          # BookFolder, BookCase (A/B/C/D classification)
│   ├── track.rs         # Audio track representation
│   ├── quality.rs       # QualityProfile (bitrate, codec, etc.)
│   ├── config.rs        # Configuration structures
│   └── result.rs        # ProcessingResult
├── utils/               # Utilities
│   ├── config.rs        # ConfigManager (load/save YAML)
│   ├── validation.rs    # DependencyChecker (FFmpeg, etc.)
│   └── sorting.rs       # Natural sorting (track1, track2, track10)
├── cli/                 # CLI interface
│   └── commands.rs      # Clap command definitions
├── audio/               # Audio operations
│   ├── ffmpeg.rs        # FFmpeg wrapper (probe, concat, transcode)
│   ├── metadata.rs      # Metadata extraction (ID3, M4A)
│   ├── chapters.rs      # Chapter generation
│   └── cue.rs           # CUE file parsing
└── core/                # Core processing logic
    ├── processor.rs     # Single book processing
    ├── parallel.rs      # Parallel batch processing
    ├── organizer.rs     # Library organization
    └── progress.rs      # Progress tracking (indicatif)
```

### Key Concepts

**BookCase Classification**:
- **Case A**: Already M4B (skip)
- **Case B**: MP3 files only (requires conversion)
- **Case C**: M4A files only (can use fast concat)
- **Case D**: Mixed MP3/M4A (requires transcode to AAC)

**Quality Profiles**: Automatically detect best quality across tracks and preserve it. Support compatibility checking for concatenation.

**Processing Modes**:
- Copy mode: Ultra-fast concatenation without re-encoding (when compatible)
- Transcode mode: Convert to AAC when necessary

---

## Code Style & Conventions

### Rust Patterns
- Use `anyhow::Result<T>` for application errors
- Use `thiserror` for custom error types
- Prefer `?` operator over explicit error handling
- Use `.context()` to add error context
- Follow Rust naming conventions (snake_case for functions/variables, PascalCase for types)

### Error Handling
```rust
// Good - provides context
let tracks = discover_tracks(&path)
    .context(format!("Failed to discover tracks in {}", path.display()))?;

// Avoid - no context
let tracks = discover_tracks(&path)?;
```

### Async/Await
- Use tokio runtime for async operations
- FFmpeg calls use `tokio::process::Command`
- Parallel processing uses `tokio::spawn` with semaphore for resource limits

### Testing
- Unit tests in same file as implementation (`#[cfg(test)] mod tests`)
- Integration tests in `tests/` directory
- Use `tempfile` for temp directories in tests
- Mock external dependencies when possible

---

## Testing Guidelines

### Running Tests
All tests should pass before committing:
```bash
cargo test --lib
cargo test --all
```

### Writing Tests
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_something() {
        // Arrange
        let input = "value";

        // Act
        let result = function_under_test(input);

        // Assert
        assert_eq!(result, expected);
    }

    #[tokio::test]
    async fn test_async_function() {
        // For async tests
    }
}
```

### Test Coverage
- All public functions should have tests
- Edge cases should be covered (empty input, invalid data, etc.)
- Quality profile comparison logic is critical (heavily tested)

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juanra/audiobook-forge](https://github.com/juanra/audiobook-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
