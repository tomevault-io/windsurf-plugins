---
trigger: always_on
description: This folder implements complete **VSDX (Microsoft Visio 2013+ XML)** file format support for draw.io. VSDX files are ZIP archives containing XML documents following the Open Packaging Conventions (OPC) standard.
---

# VSDX Import/Export Module

## Overview

This folder implements complete **VSDX (Microsoft Visio 2013+ XML)** file format support for draw.io. VSDX files are ZIP archives containing XML documents following the Open Packaging Conventions (OPC) standard.

**4 files, ~893 KB total, ~15,600 lines of code.**

| File | Lines | Size | Purpose |
|------|-------|------|---------|
| `importer.js` | 13,208 | 782 KB | VSDX/VSSX → draw.io import (JSweet-generated from Java) |
| `VsdxExport.js` | 993 | 73 KB | draw.io → VSDX export |
| `mxVsdxCanvas2D.js` | 1,153 | 30 KB | Canvas adapter capturing shape rendering as VSDX geometry |
| `bmpDecoder.js` | 287 | 9 KB | BMP image format decoder for embedded images |

---

## Architecture

```
┌─────────────────────── IMPORT ───────────────────────┐
│                                                       │
│  VSDX ZIP  →  mxVsdxCodec.decodeVsdx()               │
│                   │                                   │
│                   ├── JSZip extraction                 │
│                   ├── XML parsing (docData map)        │
│                   ├── Media extraction (mediaData map) │
│                   │     ├── EMF → PNG (server-side)    │
│                   │     ├── BMP → JPEG (BmpDecoder)    │
│                   │     └── PNG/JPEG → base64          │
│                   │                                   │
│                   ▼                                   │
│              mxVsdxModel                              │
│                   ├── Stylesheets                     │
│                   ├── Themes (mxVsdxTheme)            │
│                   ├── Masters (mxVsdxMaster)          │
│                   └── Pages (mxVsdxPage)              │
│                         └── Shapes (VsdxShape)        │
│                               ├── Geometry (Rows)     │
│                               ├── Style resolution    │
│                               └── Master inheritance  │
│                   │                                   │
│                   ▼                                   │
│              importPage() → mxGraph model → XML       │
│                                                       │
└───────────────────────────────────────────────────────┘

┌─────────────────────── EXPORT ───────────────────────┐
│                                                       │
│  mxGraph  →  VsdxExport.exportCurrentDiagrams()       │
│                   │                                   │
│                   ├── createVsdxSkeleton() (ZIP init)  │
│                   │                                   │
│                   ▼                                   │
│              convertMxModel2Page()                    │
│                   ├── convertMxCell2Shape() per cell   │
│                   │     ├── createShape() (vertices)   │
│                   │     ├── createEdge() (connectors)  │
│                   │     └── mxVsdxCanvas2D (rendering) │
│                   ├── applyMxCellStyle()              │
│                   └── addPagesXML() + addImagesRels() │
│                   │                                   │
│                   ▼                                   │
│              JSZip.generateAsync() → .vsdx download   │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## File Details

### importer.js

**Origin**: Generated from Java via **JSweet 2.0.0-rc1** transpiler. Uses nested namespaces: `com.mxgraph.io.*`.

**Namespace structure**:
```
com.mxgraph.io.mxVsdxCodec         — Main VSDX codec
com.mxgraph.io.mxVssxCodec         — Library (VSSX) codec (extends mxVsdxCodec)
com.mxgraph.io.vsdx.Shape          — Base shape wrapper
com.mxgraph.io.vsdx.VsdxShape      — VSDX-specific shape (extends Shape)
com.mxgraph.io.vsdx.mxVsdxModel    — Document model container
com.mxgraph.io.vsdx.mxVsdxPage     — Page representation
com.mxgraph.io.vsdx.mxVsdxMaster   — Master shape template
com.mxgraph.io.vsdx.mxVsdxTheme    — OOXML theme parser
com.mxgraph.io.vsdx.mxVsdxConnect  — Connection definition
com.mxgraph.io.vsdx.mxVsdxUtils    — XML/style utilities
com.mxgraph.io.vsdx.mxVsdxConstants — XML element/attribute constants
com.mxgraph.io.vsdx.mxPropertiesManager — Color palette manager
com.mxgraph.io.vsdx.mxVsdxGeometry — Single geometry section
com.mxgraph.io.vsdx.mxVsdxGeometryList — Collection of geometry sections
com.mxgraph.io.vsdx.geometry.Row   — Base geometry row
com.mxgraph.io.vsdx.geometry.*     — 16 Row subclasses
com.mxgraph.io.vsdx.theme.*        — Theme/color classes (OoxmlColor, etc.)
```

#### Key Classes

**mxVsdxCodec** — Main entry point for VSDX import.

| Method | Purpose |
|--------|---------|
| `decodeVsdx(file, callback, charset, onerror)` | Main async entry: extracts ZIP, parses XML, builds model, imports pages |
| `createMxGraph()` | Creates configured Graph instance for import |
| `importPage(page, graph, parent, noSanitize)` | Orchestrates page import: shapes → edges → layers (3-pass) |
| `processPage(graph, page)` | Encodes mxGraph model to compressed XML `<diagram>` element |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Abnerla/AI_paper](https://github.com/Abnerla/AI_paper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
