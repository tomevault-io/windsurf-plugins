---
trigger: always_on
description: **Pure UI** is a pure Dart implementation of Flutter's `dart:ui` Canvas API for non-Flutter environments.
---

# CLAUDE.md - Pure UI Developer Guide

**Pure UI** is a pure Dart implementation of Flutter's `dart:ui` Canvas API for non-Flutter environments.

## Quick Setup

```bash
dart pub get
make build
make ci        # Verify everything
```

## Essential Structure

```
lib/
├── pure_ui.dart                    # Main library entry
├── painting.dart                   # Canvas, Paint, Path, Color (~10,000 lines)
├── pure_dart_implementations.dart  # Core implementations + paragraph rendering
├── font_loader.dart                # FontLoader registry (family → TTF bytes)
├── src/text/
│   ├── ttf_parser.dart             # Binary TTF/OTF parser (cmap, glyf, hmtx, kern)
│   ├── ttf_font.dart               # TtfFont: glyph outline + advance + kerning
│   ├── font_metrics.dart           # FontMetrics (ascender, descender, unitsPerEm)
│   ├── glyph_outline.dart          # GlyphOutline / GlyphContour / GlyphPoint
│   ├── text_shaper.dart            # shapeText() → ShapedGlyph list
│   └── text_layout.dart            # layoutText() → LayoutLine list (wrap, align)
└── [other part files]

test/
├── text_rendering/                 # 8 test files for text pipeline (Phases 1–8)
└── [other test files]

tool/
└── render_japanese.dart            # Example: render Japanese text to PNG

Makefile                            # Build automation
docs/ttf_text_rendering_plan.md     # Full implementation plan (all phases complete)
```

## Key Commands

```bash
make analyze      # Lint code
make format       # Format code
make ci           # analyze + format (run before commit)
dart test         # Run tests (257 total)
make build        # Build library
make pub_publish  # Publish to pub.dev
```

## Code Style

- **Classes:** `PascalCase` → `Canvas`, `_PureDartCanvas`, `TtfFont`
- **Methods/Variables:** `camelCase` → `drawCircle()`, `strokeWidth`
- **Documentation:** `///` for public APIs
- **Formatting:** Auto via `dart format` (2-space indent)
- **Copyright headers:** Only add to files that already have Flutter team copyright. Claude-created files have no copyright header.

## Development Workflow

1. **Adding Features:** Update API in `painting.dart` → Implementation in `pure_dart_implementations.dart` → Add tests
2. **Text Rendering:** Shaper (`text_shaper.dart`) → Layout (`text_layout.dart`) → Rasterization (`pure_dart_implementations.dart`)
3. **Bug Fixes:** Create test that reproduces bug → Fix code → Verify tests pass
4. **Commits:** Run `make ci` first, then commit with descriptive message

## Core Classes

| Class | Location | Purpose |
|-------|----------|---------|
| `Canvas` | painting.dart | Main drawing API |
| `Paint` | painting.dart | Drawing style (color, width, etc.) |
| `Path` | painting.dart | Vector path with curves |
| `PictureRecorder` | painting.dart | Record drawing commands |
| `_PureDartCanvas` | pure_dart_implementations.dart | Pure Dart canvas implementation |
| `_PureDartParagraph` | pure_dart_implementations.dart | Paragraph layout + rendering |
| `_PureDartParagraphBuilder` | pure_dart_implementations.dart | Span/style accumulation |
| `FontLoader` | font_loader.dart | Font registry (family + weight/style → bytes) |
| `TtfFont` | src/text/ttf_font.dart | Parsed TTF with glyph/kerning lookup |
| `ShapedGlyph` | src/text/text_shaper.dart | Single glyph with advance, color, decoration |
| `LayoutLine` | src/text/text_layout.dart | One laid-out line with baseline + glyphs |

## Text Rendering Pipeline

```
FontLoader.load(family, bytes)          # Register TTF bytes by family name
  ↓
ParagraphBuilder(ParagraphStyle(...))   # Set font family, size, alignment, maxLines
  .pushStyle(TextStyle(...))            # Push span style (color, decoration, shadows)
  .addText("Hello")                     # Add text span
  .pop()
  .build()                              # → _PureDartParagraph
  ↓
paragraph.layout(ParagraphConstraints(width: 300))
  → shapeText()   : text → ShapedGlyph list (advance, kerning, letterSpacing)
  → layoutText()  : greedy word-wrap, hard \n breaks, maxLines, ellipsis, TextAlign
  ↓
canvas.drawParagraph(paragraph, offset)
  → Pass 1: shadows  (render glyph silhouette at shadow offset)
  → Pass 2: glyph ink (scanline fill via cached polygon subpaths)
  → Pass 3: decorations (underline / overline / line-through)
```

## Performance Caches

| Cache | Key | Avoids |
|-------|-----|--------|
| `_pureDartFontCache` | `"family_weightIdx_styleIdx"` | Re-parsing TTF bytes |
| `_glyphPolyCache` | `"fontKey/glyphId/fontSize"` | Bézier re-tessellation per render |

## Key Files by Task

| Task | File |
|------|------|
| Add drawing method | `lib/painting.dart` |
| Implement drawing logic | `lib/pure_dart_implementations.dart` |
| Add geometry type | `lib/geometry.dart` |
| Text shaping logic | `lib/src/text/text_shaper.dart` |
| Text layout / line wrapping | `lib/src/text/text_layout.dart` |
| TTF binary parsing | `lib/src/text/ttf_parser.dart` |
| Font registration | `lib/font_loader.dart` |
| Add test | `test/*.dart` or `test/text_rendering/*.dart` |
| CI/CD config | `.github/workflows/check.yml` |

## Testing

```dart
import 'dart:io';
import 'package:test/test.dart';
import 'package:pure_ui/pure_ui.dart' as ui;

void main() {
  setUpAll(() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [normidar/pure_ui](https://github.com/normidar/pure_ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
