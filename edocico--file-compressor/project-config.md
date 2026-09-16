---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A file compression/decompression utility written in Rust using the Zstandard (zstd) compression algorithm. Provides both CLI and GUI interfaces.

**Language**: CLI and library use Italian for all user-facing text (error messages, help text, comments). GUI auto-detects system locale and supports both Italian and English.

## Build and Development Commands

```bash
# Build
cargo build                    # Debug build
cargo build --release          # Release build (optimized)

# Run CLI
cargo run -- compress <file> --livello 10
cargo run -- decompress <file.zst>
cargo run -- multi-compress file1 file2 --output archive.tar.zst
cargo run -- batch "*.log" --livello 5
cargo run -- verifica <file.zst>
cargo run -- riduci scansione.pdf --max-size 2MB   # lossy, output -ridotto.pdf

# Run GUI
cargo run --bin file_compressor_gui

# Install desktop entry + binaries per-user on Linux (no sudo; --uninstall to remove)
./install-linux.sh

# Test
cargo test                     # Run all tests
cargo test -- --nocapture      # With output
cargo test <test_name>         # Specific test
cargo test -- <name1> <name2>  # Multiple filters need `--` (bare args error out)

# Lint and format
cargo clippy
cargo fmt

# Validate workflow YAML (python3-pyyaml is installed; ruby is not)
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci.yml'))"
```

**Binary outputs:**

- CLI: `target/{debug,release}/file_compressor`
- GUI: `target/{debug,release}/file_compressor_gui`

## Architecture

### Module Structure

| File | Purpose |
|------|---------|
| [src/lib.rs](src/lib.rs) | Core compression library with all compression/decompression logic |
| [src/main.rs](src/main.rs) | CLI application with progress bars |
| [src/gui.rs](src/gui.rs) | egui-based GUI application |
| [src/shrink.rs](src/shrink.rs) | "Riduci" feature: lossy in-format re-compression (PDF/JPEG/PNG/DOCX) to fit upload size limits |
| [src/updater.rs](src/updater.rs) | In-app update: GitHub release check, per-channel apply (AppImage/installer/bundle) |

### Core Library (lib.rs)

Key types:

- `CompressOptions` / `DecompressOptions` - Builder pattern for operation configuration
- `CompressOutcome` - `Compressed(CompressionResult)` or `Skipped { reason }`; skipping an already-compressed file is a normal outcome, never an `Err`
- `CompressionResult` - Stores input/output sizes
- `VerifyResult` - File integrity verification result
- `CancelFlag` (`Arc<AtomicBool>`) - Checked once per I/O block, so cancellation takes effect *within* a large file, not just between files

Key functions:

- `compress_file()` / `decompress_file()` - Single file operations with progress callbacks
- `compress_directory()` - Creates tar.zst from directory
- `compress_multiple_files()` - Bundles files into tar.zst archive; renames duplicates (`report.txt` → `report-2.txt`) since the archive is flat
- `verify_zst()` - Validates file integrity
- `declared_decompressed_size()` - Reads the original size from the zstd frame header (written by `set_pledged_src_size`), giving progress bars a real total instead of an estimate
- `estimate_compressed_size()` / `estimate_directory_compressed_size()` - Real-compression size estimates (whole file ≤4MB, three 512KB samples above; directories capped at 20 files/8MB). The GUI consumes them from a background worker with a (path, level, smart) cache — never call them on the render thread.
- `cleanup_temp_files()` - Removes in-flight temp files; call from signal handlers, which skip destructors

### CLI Commands (main.rs)

| Command | Description |
|---------|-------------|
| `compress` | Single file/directory compression (level 1-21, default 3) |
| `decompress` | Decompress .zst or extract .tar.zst |
| `multi-compress` | Create tar.zst from multiple files (clap derives kebab-case from `MultiCompress`) |
| `batch` | Compress files matching glob pattern (e.g., `*.log`, `**/*.txt`) |
| `verifica` | Verify .zst file integrity |
| `riduci` | Lossy in-format shrink of already-compressed files (`--max-size 2MB`, `--preset leggera/media/aggressiva`); output is always a new `-ridotto` file |

### Dependencies

| Crate | Purpose |
|-------|---------|
| zstd | Zstandard compression |
| clap | CLI argument parsing |
| indicatif | Progress bars |
| tar | TAR archive handling |
| rayon | Parallel batch processing |
| glob | Pattern matching for batch operations |
| eframe/rfd | GUI framework and file dialogs |
| sys-locale | GUI system locale detection |
| ctrlc | Ctrl+C signal handling for graceful interruption |
| ureq | HTTP client for update checks/downloads (blocking, rustls) |
| serde_json | GitHub API response parsing |
| semver | Version comparison for updates |
| dirs | Cross-platform state dir for update-check throttle |
| winreg | (Windows) detect Inno-installed copies via uninstall registry key |
| lopdf | PDF parsing/rewriting for the shrink feature |
| image | JPEG/PNG decode + re-encode (shrink) |
| imagequant | PNG palette quantization (shrink) |
| png | Indexed PNG writing (shrink) |
| zip | DOCX/XLSX/PPTX re-zip (shrink) |

## Design Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edocico/file_compressor](https://github.com/edocico/file_compressor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
