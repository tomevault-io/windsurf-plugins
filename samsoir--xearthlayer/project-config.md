---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**XEarthLayer** is a Rust implementation inspired by AutoOrtho, providing on-demand satellite imagery streaming for X-Plane via a FUSE virtual filesystem. It downloads satellite imagery tiles as X-Plane requests them, eliminating the need for massive pre-downloaded scenery packages.

**Core Concept**: Creates a passthrough virtual filesystem that overlays real scenery files while generating DDS textures on-demand when X-Plane requests them.

**Current Status**: Functional and tested with X-Plane 12. Scene load times reduced from 4-5 minutes to ~30 seconds with parallel tile generation.

## Design Principles

**IMPORTANT**: This project follows SOLID principles and Test-Driven Development (TDD). See `docs/dev/design-principles.md` for details.

**All code must**:
- Be developed using TDD (write tests first)
- Follow SOLID principles (traits for abstraction, dependency injection)
- Be testable in isolation with mock support
- Maintain minimum 80% test coverage (target 90%+)

## Architecture

### Implemented Components

1. **FUSE Layer** (`xearthlayer/src/fuse/fuse3/`)
   - `Fuse3PassthroughFS` - Async multi-threaded passthrough for single scenery packs
   - `Fuse3UnionFS` - Union filesystem for patch folders (priority-based merge)
   - `Fuse3OrthoUnionFS` - Consolidated FUSE mount for all ortho sources (patches + packages)
   - All FUSE operations run asynchronously on the Tokio runtime
   - Shared traits: `FileAttrBuilder`, `DdsRequestor` (SOLID: Interface Segregation)
   - Pattern matching for DDS files: `{row}_{col}_ZL{zoom}.dds`

2. **Tile Generation** (`xearthlayer/src/tile/`)
   - `TileGenerator` trait with dependency injection
   - `DefaultTileGenerator` - Download + encode pipeline
   - `ParallelTileGenerator` - Thread pool with request coalescing

3. **Download Orchestrator** (`xearthlayer/src/orchestrator/`)
   - Parallel download of 256 chunks (16×16) per tile
   - Configurable concurrency, timeout, and retry logic
   - 80% minimum success rate requirement

4. **DDS Compression** (`xearthlayer/src/dds/`)
   - BC1/BC3 (DXT1/DXT5) compression
   - 5-level mipmap chain generation
   - `ImageCompressor` trait for single-level backends:
     - `SoftwareCompressor` — Pure-Rust fallback
     - `IspcCompressor` — SIMD-optimized via Intel ISPC (default)
   - `MipmapCompressor` trait for full-pipeline backends (GPU):
     - `GpuEncoderChannel` — Worker-side mipmap streaming, zero-clone channel transfer
   - `MipmapStream` — Memory-efficient iterator yielding one mipmap level at a time (no clones)
   - GPU encoding architecture: `mpsc` channel → dedicated pipeline worker → `GpuBlockCompressor`
   - `WgpuCompressor` wraps `block_compression` crate (ISPC kernels ported to WGSL compute shaders)
   - Worker-side tile processing: one channel round-trip per tile, GPU buffer reuse across mipmap levels
   - `create_gpu_resources()` — shared factory for device/queue/compressor creation (DRY)
   - GPU pipeline hardening:
     - `map_async` errors propagated via `std::sync::mpsc` (not silently ignored)
     - Worker panic recovery via `catch_unwind` (logs error, sends failure, drains queue, breaks loop)
     - `device.on_uncaptured_error()` registered for device loss logging
     - Structured tracing on buffer lifecycle and error paths

5. **Cache System** (`xearthlayer/src/cache/`, `xearthlayer/src/service/cache_layer.rs`)
   - Three-tier cache hierarchy: Memory → DDS Disk → Chunk Disk → Network
   - `CacheLayer` - Service-owned three-tier cache lifecycle with budget management
   - `CacheService` - Self-contained cache with internal GC daemons
   - `MemoryCacheProvider`: moka-based LRU eviction (default 512MB, staging buffer)
   - `DiskCacheProvider`: Two instances — DDS tiles and raw chunks, each with own GC
   - DDS disk cache: Encoded DDS tiles on disk, avoids re-encoding on memory eviction (~3.5ms NVMe read vs ~50-200ms re-encode)
   - Shared disk budget: `cache.disk_size` split by `cache.dds_disk_ratio` (default 0.6 = 60% DDS, 40% chunks)
   - DDS disk directory: `{cache_dir}/{provider}/dds/`, chunks keep existing provider directory
   - Region-based disk layout: files in 1°×1° DSF subdirs (e.g., `+33-119/tile_15_12754_5279.cache`)
   - Read path: memory hit → DDS disk hit (+ promote to memory) → chunk disk (assemble+encode) → network
   - Write path: After encode, fire-and-forget to both memory AND DDS disk
   - Parallel startup scanning via rayon over region directories
   - `migrate_cache()` for migrating flat-layout caches to region layout
   - Bridge adapters for executor integration (MemoryCacheBridge, DdsDiskCacheBridge, DiskCacheBridge)
   - Per-provider cache directories
   - `XEarthLayerService::start()` creates `CacheLayer` with proper metrics ordering

6. **Configuration** (`xearthlayer/src/config/`)
   - INI file at `~/.xearthlayer/config.ini`
   - Auto-detection of X-Plane installation
   - See `docs/configuration.md` for all settings

7. **Providers** (`xearthlayer/src/provider/`)
   - Bing Maps (free, no API key)
   - Google Maps GO2 (free, no API key)
   - Google Maps (paid, requires API key)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samsoir/xearthlayer](https://github.com/samsoir/xearthlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
