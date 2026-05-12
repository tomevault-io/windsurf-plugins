---
trigger: always_on
description: Enables sorting by capture date in file explorers.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dual-implementation Snapchat Memories downloader with web and Python versions that parse `memories_history.html` exports to download media with metadata preservation.

**Web Version**: `docs/index.html` (2743 lines) - Client-side processing with FFmpeg.wasm for video overlays
**Python Version**: `download_memories.py` (1608 lines) - CLI tool with FFmpeg subprocess for video overlays

Both versions share identical architecture but differ in implementation details and feature sets.

## Development Commands

### Python Setup
```bash
./setup.sh                    # Create venv, install dependencies
source venv/bin/activate      # Activate environment
python3 download_memories.py --test  # Quick test (first 3 files)
deactivate                    # Exit venv
```

### Common Python Usage
```bash
# Basic download
python3 download_memories.py

# Resume interrupted download
python3 download_memories.py --resume

# Retry only failed items
python3 download_memories.py --retry-failed

# Merge overlays (requires FFmpeg for videos)
python3 download_memories.py --merge-overlays

# Timestamp-based naming instead of sequential
python3 download_memories.py --timestamp-filenames

# Process only videos with overlay merging
python3 download_memories.py --videos-only --merge-overlays

# Custom output directory
python3 download_memories.py -o ./my-memories
```

### Web Development
No build process - edit `docs/index.html` directly. FFmpeg.wasm artifacts are auto-synced via GitHub Actions when `package.json` dependencies update.

To manually sync FFmpeg artifacts:
```bash
npm install
# Copy from node_modules/@ffmpeg/ to docs/ffmpeg/
```

## Core Architecture

### Dual Implementation Pattern
Both versions implement the same workflow but with different runtime environments:
- **Web**: Client-side browser processing (privacy-first, no upload)
- **Python**: CLI processing (advanced features, FFmpeg CLI, full EXIF)

### Data Flow
1. Parse `memories_history.html` → extract URLs, dates, GPS coordinates
2. Initialize metadata.json with all memories as "pending"
3. For each memory:
   - Download from Snapchat URL
   - Detect ZIP files (magic bytes `PK`) containing overlays
   - Check MD5 hash for duplicates (during download, not post-process)
   - Extract `-main` and `-overlay` files from ZIP or save single file
   - Add EXIF metadata to images (GPS + timestamp)
   - Optionally merge overlays (instant for images, 1-5min for videos)
   - Set file modification time to capture date
   - Update metadata state: `pending` → `in_progress` → `success`/`failed`
4. Optional post-processing: join multi-snap videos, deduplicate

### Metadata State Machine
`metadata.json` tracks download progress enabling crash recovery:
- **States**: `pending`, `in_progress`, `success`, `failed`, `skipped`
- **Saved after every download** for resume capability
- **Same format** works for both web and Python versions
- Stores file paths, sizes, types (main/overlay/merged/single/duplicate)

### Overlay Processing Architecture

**Two strategies based on media type:**

**Images** (fast, instant):
- Python: Pillow library for alpha compositing
- Web: Canvas API for alpha compositing
- Preserves original format (JPEG, PNG, WebP, GIF, BMP, TIFF)

**Videos** (slow, 1-5 minutes each):
- Python: FFmpeg subprocess with filter chains
- Web: FFmpeg.wasm (WebAssembly port)
- FFmpeg filter: `[1:v]scale=WxH[ovr];[0:v][ovr]overlay=shortest=1[outv]`
- Handles both image overlays (with `-loop 1`) and video overlays

**Deferred Processing Pattern** (`--defer-video-overlays`):
- Downloads all content first (main + overlay files saved separately)
- Processes video merges at end in batch
- Prevents memory buildup during initial downloads
- Web version: separate ZIP per deferred video

### Memory Management (Web Only)
FFmpeg.wasm has memory leak issues, so the web version implements:
- **Auto-reset**: Terminates and reinitializes FFmpeg every N videos (5/10/20/unlimited)
- **Memory monitoring**: Chrome `performance.memory` API warns at >60% usage
- **GC pauses**: 2-second pause between deferred video processing for garbage collection

### Duplicate Detection
Runs **during download** (not post-processing) to save bandwidth:
1. Check file size match
2. Compute MD5 hash of new download
3. Compare with existing files
4. Skip download if duplicate found

Python also has `--remove-duplicates` for retroactive scanning.

### Multi-Snap Joining (Python Only)
Detects videos within 10-second time windows (indicates multi-snap stories):
- Uses FFmpeg concat demuxer: `-f concat -safe 0 -i concat.txt -c copy`
- No re-encoding (fast, lossless)
- Deletes originals after successful join

## Key Technologies

**Python Dependencies**:
- `requests` - HTTP downloads with custom User-Agent
- `Pillow` - Image overlay compositing, format preservation
- `piexif` - EXIF metadata encoding (GPS + dates in JPEG/PNG/WebP)
- `subprocess` - FFmpeg process invocation
- `zipfile` - Extract overlay ZIP files
- `hashlib` - MD5 for duplicate detection

**Web Dependencies**:
- `JSZip` 3.10.1 - ZIP file handling
- `piexifjs` 1.0.6 - EXIF for JPEG only (browser limitation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrefecto/Snapchat-Memories-Downloader](https://github.com/andrefecto/Snapchat-Memories-Downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
