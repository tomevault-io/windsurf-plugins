---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KiDoom is a technical demonstration that runs DOOM within KiCad's PCBnew using real PCB traces and footprints as the rendering medium. The project features a complete triple-mode rendering system: SDL window for gameplay, Python wireframe renderer for reference, and KiCad PCB traces for the technical demonstration.

**Key Innovation:** Instead of raster pixel rendering (64,000 pixels = unworkable), this uses vector rendering with PCB traces (100-300 line segments per frame) and real component footprints for entities, providing a 200-500x performance improvement.

**Entity Innovation:** Entities (enemies, items, decorations) are rendered as real PCB footprints with component complexity matching gameplay importance:
- **Collectibles** (health, ammo) → SOT-23 (3-pin small packages)
- **Decorations** (barrels, bodies) → SOIC-8 (8-pin flat packages)
- **Enemies** (zombies, demons) → QFP-64 (64-pin complex packages)

**Expected Performance:** 10-25 FPS in KiCad, 60+ FPS in standalone renderer

## Architecture

### Triple-Mode Rendering System

**Mode 1: SDL Window (Gameplay)**
- Full DOOM graphics for keyboard input and reference
- Standard SDL2 rendering
- Position: (0, 420) on screen (below Python renderer)

**Mode 2: Python Wireframe Renderer**
- Standalone pygame-based wireframe visualization
- Displays extracted vectors from DOOM engine
- Position: (0, 0) on screen

**Mode 3: KiCad PCB Rendering**
- Real PCB traces for walls
- Real component footprints for entities
- Electrically authentic design

All three modes receive identical vector data from DOOM via Unix socket.

### Core Components

**DOOM Engine (C):**
- Uses doomgeneric framework (designed for porting DOOM to new platforms)
- Dual-mode operation: SDL rendering + vector extraction
- Located in `doom/source/doomgeneric_kicad_dual_v2.c`
- Compiled binary: `doom/doomgeneric_kicad`
- Custom patches to vissprite_t for entity type extraction

**KiCad Plugin (Python):**
- Main entry point: `doom_plugin_action.py` (ActionPlugin with file logging)
- PCB renderer: `pcb_renderer.py` (wireframe edges + footprint placement)
- Communication bridge: `doom_bridge.py` (two-phase socket server)
- Entity types: `entity_types.py` (150+ DOOM entities categorized)
- Object pools: `object_pool.py` (pre-allocated PCB objects by category)
- Coordinate transform: `coordinate_transform.py` (DOOM pixels → KiCad nm with A4 centering)

### Communication Protocol

Binary protocol over Unix domain socket (`/tmp/kicad_doom.sock`):
```
[4 bytes: message type][4 bytes: payload length][N bytes: JSON payload]
```

Message types:
- 0x01: FRAME_DATA (DOOM → Python)
- 0x02: KEY_EVENT (Python → DOOM)
- 0x03: INIT_COMPLETE (Python → DOOM)
- 0x04: SHUTDOWN (bidirectional)

### PCB Element Mapping

| DOOM Element | PCB Element | Specification | Visual Meaning |
|-------------|-------------|---------------|----------------|
| Wall segments | `PCB_TRACK` (wireframe) | F.Cu/B.Cu, width encodes distance | Blue traces, thick=close |
| Collectibles | `FOOTPRINT` (SOT-23) | 3-pin small package | Health, ammo, keys |
| Decorations | `FOOTPRINT` (SOIC-8) | 8-pin flat package | Barrels, bodies, props |
| Enemies | `FOOTPRINT` (QFP-64) | 64-pin complex package | Zombies, demons, player |
| Projectiles | `PCB_VIA` | Drilled holes | Bullets, fireballs |
| HUD elements | `PCB_TEXT` | F.SilkS silkscreen | Health, ammo counters |

**Wall Rendering:** Wireframe edges (4 traces per wall: top, bottom, left, right)
- All walls use B.Cu (blue) layer
- Distance encoded as trace width (thick = close, thin = far)

**Entity Rendering:** Real PCB footprints based on DOOM entity type (MT_* enum)
- Type extracted from vissprite_t.mobjtype (custom DOOM patch)
- Categorized by gameplay significance
- Pre-loaded from KiCad standard libraries

## Development Commands

### Building DOOM Engine

```bash
# Automated build (recommended)
cd doom/source
./build.sh

# Manual build
git clone https://github.com/ozkl/doomgeneric.git
cd doomgeneric/doomgeneric

# Apply KiDoom patches to doomgeneric source
# Patch 1: Add mobjtype field to vissprite_t structure
sed -i '' '/lighttable_t.*colormap;/a\
\
    // KiDoom: Store entity type for footprint selection\
    int			mobjtype;\
' r_defs.h

# Patch 2: Capture entity type during vissprite creation
sed -i '' '/vis->mobjflags = thing->flags;/a\
    vis->mobjtype = thing->type;  // KiDoom: Capture entity type\
' r_things.c

# Copy platform files
cp /path/to/KiDoom/doom/source/doomgeneric_kicad_dual_v2.c .
cp /path/to/KiDoom/doom/source/doom_socket.c .
cp /path/to/KiDoom/doom/source/doom_socket.h .
cp /path/to/KiDoom/doom/source/Makefile.kicad_dual .

# Build
make -f Makefile.kicad_dual

# Copy binary and WAD back to plugin
cp doomgeneric_kicad_dual /path/to/KiDoom/doom/doomgeneric_kicad
cp doom1.wad /path/to/KiDoom/doom/
```

### Running Tests

**Standalone Renderer Test:**
```bash
# Terminal 1: Start Python wireframe renderer
./run_standalone_renderer.py

# Terminal 2: Launch DOOM
./run_doom.sh dual -w 1 1  # E1M1 with both SDL + vectors
```

**KiCad Plugin Test:**
```bash
# In KiCad:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MichaelAyles/KiDoom](https://github.com/MichaelAyles/KiDoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
