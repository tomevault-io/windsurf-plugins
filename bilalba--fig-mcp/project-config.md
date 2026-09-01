---
trigger: always_on
description: An MCP (Model Context Protocol) server for parsing `.fig` files. This enables AI assistants to understand and extract design information from the `.fig` file format for implementation guidance.
---

# Fig MCP Server

## Project Overview

An MCP (Model Context Protocol) server for parsing `.fig` files. This enables AI assistants to understand and extract design information from the `.fig` file format for implementation guidance.

## Architecture

### Core Components

1. **Parser** (`src/parser/`)
   - `fig-reader.ts` - ZIP archive extraction using Central Directory (handles data descriptors)
   - `kiwi-parser.ts` - Binary kiwi format parsing with deflate + zstd decompression
   - `layout-inference.ts` - Converts raw node data to structured layout information
   - `types.ts` - TypeScript type definitions for `.fig` document structure

2. **MCP Server** (`src/mcp/`)
   - `server.ts` - MCP server implementation with tools for querying fig files

3. **Utilities**
   - `inspect-fig.ts` - CLI tool for inspecting fig files during development
   - `test-parser.ts` - Local tool-flow test harness for image tools

4. **Renderer** (`src/renderer/`)
   - `render-screen.ts` - Main SVG renderer for node subtrees
   - `render-types.ts` - TypeScript types for rendering
   - `render-utils.ts` - Transform, path building, and XML utilities
   - `paint-utils.ts` - Paint/fill/stroke handling
   - `vector-renderer.ts` - Vector path decoding and rendering
   - `screenshot.ts` - SVG to PNG conversion via resvg
   - `index.ts` - Public exports for the renderer module

5. **Web Viewer** (`src/web-viewer/`)
   - `server.ts` - HTTP server wrapping existing parser/renderer
   - `build-client.ts` - esbuild bundler for client code
   - `client/` - Browser-based UI (index.html, styles.css, viewer.ts)

## File Format Details

### .fig Archive Structure
`.fig` files are ZIP archives. **Important**: They use data descriptors (flag bit 3), so sizes must be read from Central Directory, not local headers.

Contents:
- `canvas.fig` - Main document data (kiwi binary format, compressed)
- `meta.json` - File metadata (name, background color, etc.)
- `thumbnail.png` - Preview image
- `images/` - Image assets (hash-named files)
- `videos/` - Video assets (if any)

### canvas.fig Binary Format
```
[header: "fig-kiwi" (8 bytes)]
[version: uint32 LE (4 bytes)]
[schema_compressed_length: uint32 LE (4 bytes)]
[schema_compressed: deflate-raw compressed binary kiwi schema]
[data_compressed_length: uint32 LE (4 bytes)]
[data_compressed: zstd compressed document data]
```

**Compression:**
- Schema chunk: deflate-raw (use `pako.inflateRaw`)
- Data chunk: zstd (use `fzstd.decompress`) - detected by magic `0xFD2FB528`

### Kiwi Schema (v101)

The schema has **530 definitions** including:

**Key Enums:**
- `NodeType`: DOCUMENT, CANVAS, FRAME, GROUP, TEXT, RECTANGLE, ELLIPSE, VECTOR, COMPONENT, INSTANCE, etc.
- `BlendMode`: NORMAL, MULTIPLY, SCREEN, OVERLAY, etc.
- `PaintType`: SOLID, GRADIENT_LINEAR, GRADIENT_RADIAL, IMAGE, VIDEO, etc.

**Key Messages:**
- `Message`: Root message with `nodeChanges: NodeChange[]`
- `NodeChange`: Contains ALL node properties (guid, type, name, size, transform, fills, strokes, effects, text data, layout, etc.)

### Image Storage

Images are stored in the `images/` folder with SHA-1 hash filenames (40 hex chars).

**fillPaints with type "IMAGE":**
```typescript
{
  type: "IMAGE",
  image: {
    hash: { "0": 225, "1": 2, ..., "19": 90 },  // 20 bytes = SHA-1
    name: "image"
  },
  imageThumbnail: {
    hash: { "0": 35, "1": 57, ..., "19": 191 },  // Thumbnail version
    name: "image"
  },
  originalImageWidth: 993,
  originalImageHeight: 4096,
  imageScaleMode: "FILL",  // or "FIT", "TILE", etc.
  scale: 0.5,
  rotation: 0
}
```

**Hash to filename conversion:**
```typescript
function hashBytesToHex(hash: Record<string, number>): string {
  const bytes = [];
  for (let i = 0; i < 20; i++) bytes.push(hash[String(i)]);
  return bytes.map(b => b.toString(16).padStart(2, '0')).join('');
}
// Result: "e10220e0eb8a423d480ad3937e0efa004848af5a"
```

### Parsing Details

**Fixed Issues:**
1. Root type detection - Was finding "NodeChange" before "Message" in schema definitions. Fixed by searching for specific types ("Message", "Document", etc.) in priority order.
2. Tree reconstruction - NodeChanges is a flat array. Each node has a `parentIndex.guid` that references its parent. Implemented `buildTreeFromNodeChanges()` to rebuild the tree hierarchy.

## Key Commands

```bash
# Development
npm run dev        # Start MCP server in watch mode
npm run build      # Build TypeScript to dist/

# Web Viewer
npm run viewer /path/to/design.fig    # Start viewer with a file
npm run viewer                         # Start viewer (open file via UI)
npm run viewer:dev                     # Dev mode (watch for client changes)
# Then open http://localhost:3000

# Testing/Inspection
npm test                                # Image tool checks (skips if files missing)
npx tsx src/inspect-fig.ts <file.fig> list      # Archive contents
npx tsx src/inspect-fig.ts <file.fig> schema    # Kiwi schema (530 defs)
npx tsx src/inspect-fig.ts <file.fig> summary   # Document structure
npx tsx src/inspect-fig.ts <file.fig> raw       # Raw decoded message
npx tsx src/inspect-fig.ts <file.fig> stats     # Node type counts
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bilalba/fig-mcp](https://github.com/bilalba/fig-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
