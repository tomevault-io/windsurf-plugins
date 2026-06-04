---
trigger: always_on
description: Reverse engineering project for extracting and visualizing maps and 3D objects from **Test Drive III: The Passion** (1990 DOS game by Accolade).
---

# Test Drive 3 Maps - Reverse Engineering Project

Reverse engineering project for extracting and visualizing maps and 3D objects from **Test Drive III: The Passion** (1990 DOS game by Accolade).

## Overview

This project parses the binary data files from Test Drive 3 to extract:
- Race track maps (Pacific Coast and Cape Cod courses)
- 3D tile geometry (terrain, roads, scenery)
- 3D objects (signs, vehicles, buildings, trains, etc.)
- Player cars (Corvette, Diablo, NSX, Stealth, Mythos)
- The game was written in Microsoft C 5.1

Includes a Three.js-based online viewer and exports to Wavefront OBJ format.

## Project Structure

```
Test-Drive-3-Maps/
├── src/
│   ├── browser/        # Browser-only modules (TypeScript)
│   │   ├── main.ts     # Browser entry point
│   │   └── scene.ts    # Three.js scene setup
│   ├── shared/         # Browser + Node shared modules (TypeScript)
│   │   ├── extract.ts  # Binary parser for TD3 formats
│   │   ├── mapgen.ts   # Map assembly logic
│   │   ├── objects.ts  # Object loading/categorization
│   │   ├── color.ts    # Color mapping extraction
│   │   ├── files.ts    # File loading and storage (universal)
│   │   ├── types.ts    # Shared type definitions
│   │   ├── lzw.ts      # LZW decompression for VGA images
│   │   └── rle.ts      # RLE unpacking for VGA images
│   └── tools/          # CLI tools (Node.js, TypeScript)
│       ├── export/     # OBJ exporter tool
│       │   ├── toobj.ts          # CLI export script
│       │   └── toWaveFrontObj.ts # OBJ/MTL writer
│       ├── imgviewer/  # VGA image extractor tool
│       │   └── imgviewer.ts      # CLI tool to extract images from DAT files
│       └── lstviewer/  # LST file viewer tool
│           └── lstviewer.ts      # CLI viewer for LST files
├── public/
│   └── base/           # Game data files (DAT/POB)
├── objs/               # Exported Wavefront OBJ files
├── spec/               # File format specifications
│   ├── 3d-object-format.md
│   ├── dat-file-layouts.md
│   ├── lst-file-format.md
│   ├── map-format.md
│   └── vga-image-format.md
├── index.html          # Vite entry point
├── vite.config.js      # Vite configuration
├── tsconfig.json       # TypeScript configuration
└── package.json
```

## Game Data Files (public/base/)

Original game data files required for extraction:

| File          | Description                                                                              |
|---------------|------------------------------------------------------------------------------------------|
| `SCENE01.DAT` | Pacific Coast course data (5 maps, tiles, palettes)                                      |
| `SCENE01.LST` | Resource index for SCENE01.DAT (scene variant, 1638 bytes)                               |
| `SCENE02.DAT` | Cape Cod course data (5 maps, tiles, objects)                                            |
| `SCENE02.LST` | Resource index for SCENE02.DAT (scene variant, 1638 bytes)                               |
| `DATAB.DAT`   | Shared tiles, objects, menu graphics                                                     |
| `DATAC.DAT`   | Color palettes                                                                           |
| `C*.DAT`      | Car data files (CCERV, CCNSX, CDIAB, CMYTH, CSTEL)                                       |
| `C*.LST`      | Car resource index files (car variant, 675 bytes each)                                   |
| `C*.POB`      | Player car models (CCERV=Corvette, CCNSX=NSX, CDIAB=Diablo, CMYTH=Mythos, CSTEL=Stealth) |

### LST Files

LST files are binary resource indices that provide offsets and sizes for resources within corresponding DAT files. Two variants exist:

| LST File    | DAT File    | Variant | Content                   |
|-------------|-------------|---------|---------------------------|
| SCENE01.LST | SCENE01.DAT | Scene   | Pacific Coast course data |
| SCENE02.LST | SCENE02.DAT | Scene   | Cape Cod course data      |
| CCERV.LST   | CCERV.DAT   | Car     | Corvette CERV III         |
| CCNSX.LST   | CCNSX.DAT   | Car     | Acura NSX                 |
| CDIAB.LST   | CDIAB.DAT   | Car     | Lamborghini Diablo        |
| CMYTH.LST   | CMYTH.DAT   | Car     | Pininfarina Mythos        |
| CSTEL.LST   | CSTEL.DAT   | Car     | Stealth R/T Turbo         |

See [`spec/lst-file-format.md`](spec/lst-file-format.md) for detailed format documentation.

## Source Files

### Browser (src/browser/)
| File             | Description                                                     |
|------------------|-----------------------------------------------------------------|
| `main.ts`        | Browser entry point. Loads data files, builds maps, handles UI. |
| `scene.ts`       | Three.js scene setup (camera, lights, renderer).                |

### Shared (src/shared/)
| File         | Description                                                                    |
|--------------|--------------------------------------------------------------------------------|
| `extract.ts` | Binary parser for TD3 file formats. Decodes vertices, polygons.                |
| `mapgen.ts`  | Map assembly - positions tiles and objects in world space.                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s-macke/Test-Drive-3-Maps](https://github.com/s-macke/Test-Drive-3-Maps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
