---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rusty Backup is a cross-platform GUI application (Rust, egui/eframe) for backing up and restoring vintage computer hard disk images with partition resizing and CHD compression. Target users are retro computing enthusiasts backing up CF/SD cards from DOS, Windows 9x, and early Linux systems.

Licensed under AGPL-3.0. The full specification lives in `PROJECT-SPEC.md`.

## Coding Style
Read CONTRIBUTING.md prior to making code changes in this repository.

## Build Commands

```bash
cargo build                    # Debug build (version: x.y.z-dev)
cargo build --release          # Release build (version from Cargo.toml or RELEASE_VERSION env)
cargo run                      # Run (debug)
cargo run --release            # Run (release)
cargo test                     # All tests
cargo test test_name           # Single test
cargo test --lib               # Unit tests only
cargo test --test '*'          # Integration tests only
cargo fmt --check              # Check formatting
cargo fmt                      # Auto-format
cargo clippy                   # Lint
```

## Build Infrastructure

### Versioning
- Version is set at compile time via `APP_VERSION` environment variable in `build.rs`
- Debug builds: `{version}-dev` (e.g., `0.1.0-dev`)
- Release builds: `RELEASE_VERSION` env var or falls back to `CARGO_PKG_VERSION`
- CI/CD uses date-based versioning: `YYYYMMDDHHMM` format

### Cross-Platform Icons
- Icons are embedded at compile time from `assets/icons/`
- Windows: `icon.ico` embedded in exe resources via winres
- macOS: `icon.icns` included in .app bundle
- Linux: `icon-256.png` for window and desktop integration
- Icon generation script: `scripts/generate-icon.sh` (requires ImageMagick)

### Platform-Specific Builds
- Windows: Console window hidden in release builds (`windows_subsystem = "windows"`)
- macOS: Built as .app bundle with Info.plist, packaged as DMG
- Linux: AppImage format with .desktop file integration

### CI/CD Pipeline
- GitHub Actions workflow: `.github/workflows/release.yml`
- Builds for: Windows (x86/x64), macOS (arm64/x64), Linux (x64 AppImage)
- Auto-release on push to main/master with generated version tag
- Artifacts: ZIP (Windows), DMG (macOS), AppImage (Linux)

### Update Checking
- Optional automatic update detection from GitHub releases
- Configure via `config.json` (see `config.json.example`)
- GUI shows update notification banner when newer version available
- Module: `src/update.rs`

## Architecture

The codebase separates GUI from core logic with these major modules:

- **`gui/`** - egui/eframe UI layer with tabs (Backup, Restore, Inspect), progress indicators, log panel, version display, and update notifications. Long-running disk operations run on separate threads to keep the UI responsive.
- **`partition/`** - MBR and GPT partition table parsing, export, alignment detection, `PartitionSizeOverride` for resize/restore, and `patch_mbr_entries`/`lba_to_chs` for MBR patching.
- **`fs/`** - Trait-based filesystem abstraction (`Filesystem` trait in `filesystem.rs`, `FileEntry` in `entry.rs`). FAT12/16/32 implementation in `fat.rs` includes browsing, `CompactFatReader` for smart compaction, and in-place resize/validation/BPB patching. Factory functions in `mod.rs` route by partition type byte. See `src/fs/README.md`.
- **`rbformats/`** - Output format handlers (VHD, CHD, Zstd, Raw) with compress/decompress APIs, plus `reconstruct_disk_from_backup` for restore and VHD export. Each format in its own file. See `src/rbformats/README.md`.
- **`backup/`** - Backup orchestration (`run_backup`), `CompressionType` enum, folder structure management, JSON metadata serialization, and checksum verification (CRC32 or SHA256).
- **`restore/`** - Restore orchestration with flexible partition sizing (original, minimum, custom) and alignment preservation.
- **`update/`** - Update checking against GitHub releases, configuration via `config.json`, and update notification UI.
- **`error.rs`** - Centralized error types using `thiserror`, with `anyhow::Result` for propagation.

### Backup Format

Each backup is a folder. Two layouts depending on the chosen output:

**Per-partition** (Zstd / Raw / per-partition VHD):
- `metadata.json` - partition info, alignment data, checksums, bad sectors
- `mbr.bin` / `mbr.json` or `gpt.json` - partition table exports
- `partition-N.<ext>` - one compressed file per partition (`.zst`, `.raw`, `.vhd`)
- Per-file checksum files (`.sha256` or `.crc32`)

**Single-file CHD** (CHD output):
- `metadata.json` with `layout: "single-file-chd"` and per-partition
  `offset_in_disk` byte ranges instead of per-file references
- `mbr.json` / `gpt.json` / `apm.json` - parsed partition-table sidecar (raw
  bytes live inside the CHD)
- `<backup-name>.chd` - one disk image with table at sector 0, partitions
  at their declared offsets, gaps zero-filled. `chdman info` opens it,
  MAME loads it.

CHD output never produces per-partition CHDs — the single-file layout is
the only CHD shape rusty-backup writes.

### Key Design Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danifunker/rusty-backup](https://github.com/danifunker/rusty-backup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
