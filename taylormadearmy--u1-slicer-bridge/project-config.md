---
trigger: always_on
description: This repo is intended to be built with an AI coding agent (Claude Code in VS Code). Treat this document as binding.
---

# AGENTS.md — AI Coding Agent Operating Manual (u1-slicer-bridge)

This repo is intended to be built with an AI coding agent (Claude Code in VS Code). Treat this document as binding.

---

## Project purpose

Self-hostable, Docker-first service for Snapmaker U1:

upload `.3mf` → validate plate → slice with Snapmaker OrcaSlicer → preview → print via Moonraker.

**Current Status:** Fully functional upload-to-print workflow with printer status monitoring.

---

## Non-goals (v1)

- No MakerWorld scraping (M26 adds optional authenticated link import)
- No per-object filament assignment (single filament per plate)
- No mesh repair or geometry modifications
- No multi-material/MMU support
- LAN-first by default (no cloud dependencies)

---

## Milestones Status

### Foundation & Core Pipeline
✅ M0 skeleton - Docker, FastAPI, services  
✅ M1 database - PostgreSQL with uploads, jobs, filaments  
✅ M3 object extraction - 3MF parser (handles MakerWorld files)  
✅ M4 ~~normalization~~ → plate validation - Preserves arrangements  
✅ M5 ~~bundles~~ → direct slicing with filament profiles  
✅ M6 slicing - Snapmaker OrcaSlicer v2.2.4, Bambu support  

### Device Integration & Print Execution
✅ M2 moonraker - Health check, configurable URL, print status polling
✅ M8 print control - Send to printer, pause/resume/cancel, printer status page

### File Lifecycle & Job Management
✅ M9 sliced file access - Browse and view previously sliced G-code files  
✅ M10 file deletion - Delete old uploads and sliced files  

### Multi-Plate & Multicolour Workflow
✅ M7.1 multi-plate support - Multi-plate 3MF detection and selection UI  
✅ M11 multifilament support - Color detection from 3MF, auto-assignment, manual override, multi-extruder slicing  
✅ M15 multicolour viewer - Show color legend in viewer with all detected/assigned colors  
✅ M16 flexible filament assignment - Allow overriding color per extruder (separate material type from color)
✅ M17 prime tower options - Add configurable prime tower options for multicolour prints
✅ M18 multi-plate visual selection - Show plate names and preview images when selecting plates

### Preview & UX
✅ M7 preview - Interactive G-code layer viewer (superseded by M12 3D viewer)
✅ M12 3D G-code viewer - Interactive 3D preview using gcode-preview + Three.js (orbit rotation, multi-color, arc support, build volume)
✅ M20 G-code viewer zoom - Zoom in/out buttons, scroll-wheel zoom toward cursor, click-drag pan, fit-to-bed reset
✅ M21 upload/configure loading UX - Progress indicators during upload preparation
✅ M22 navigation consistency - Standardized actions across UI
✅ M28 printer status page - Always-accessible printer status overlay with live progress, temps, and controls

### Slicing Controls & Profiles
✅ M7.2 build plate type & temperature overrides - Set bed type per filament and override temps at slice time
✅ M13 custom filament profiles - Import/export OrcaSlicer JSON profiles with advanced slicer settings passthrough
✅ M24 extruder presets - Preconfigure default slicing settings and filament color/type per extruder
❌ M19 slicer selection - Choose between OrcaSlicer and Snapmaker Orca for slicing
✅ M23 common slicing options - Wall count, infill pattern/density, supports, brim, skirt
✅ M29 3-way setting modes - Per-setting model/orca/override with file print settings detection

### Performance
✅ M25 API performance - Metadata caching at upload, async slicing (asyncio.to_thread), batch 3MF reads, profile caching
✅ M27 concurrency hardening - UUID temp files in profile_embedder, asyncio.Semaphore caps concurrent slicer processes (configurable MAX_CONCURRENT_SLICES)

### Platform Expansion
❌ M14 multi-machine support - Support for other printer models beyond U1
✅ M26 MakerWorld link import - Paste a MakerWorld URL to preview model info and download 3MF. Optional feature (off by default), cookie auth for unlimited downloads, browser-like request headers
✅ M30 STL upload support - Accept .stl files via trimesh STL→3MF wrapper. Single-filament only (no multi-plate, no color detection, no embedded print settings). OrcaSlicer slices the wrapped 3MF as normal.
❌ M31 Android companion app - Lightweight WebView wrapper (~50 lines Kotlin, ~1-2MB APK). Provides standalone app launch (no browser chrome), share target for MakerWorld URLs, configurable server IP. Works over plain HTTP on LAN. Built via GitHub Actions, distributed as APK from Releases.

### Build Plate & Workflow Enhancements
✅ M32 Multiple copies - Grid layout engine for duplicating objects on the build plate (1-100 copies, auto spacing, metadata patching for Orca compatibility)
❌ M33 Move objects on build plate - Interactive drag-to-position objects before slicing
✅ M34 Vertical layer slider - Side-mounted vertical range input for G-code layer navigation
✅ M35 Settings backup/restore - Export/import all settings (filaments, presets, defaults) as portable JSON
❌ M36 AI-powered model colorization - Segment single-color models into geometric regions, assign colors (manual + Claude Vision AI suggestions), output paint_color 3MF for multi-color printing. Depends on M33 (shared 3D viewer). See `memory/milestone-ai-colorization.md`

**Current:** 34 / 39 complete (87%)

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taylormadearmy/u1-slicer-bridge](https://github.com/taylormadearmy/u1-slicer-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
