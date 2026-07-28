---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build
```bash
dotnet build UglyToad.PdfPig.Rendering.Skia.sln
# Or just the main library:
dotnet build UglyToad.PdfPig.Rendering.Skia/UglyToad.PdfPig.Rendering.Skia.csproj -c Release
```

### Test
```bash
# All tests
dotnet test UglyToad.PdfPig.Rendering.Skia.sln

# Single test class or method
dotnet test --filter "FullyQualifiedName~ClassName" UglyToad.PdfPig.Rendering.Skia.sln
dotnet test --filter "FullyQualifiedName~ClassName.MethodName" UglyToad.PdfPig.Rendering.Skia.sln
```

Test framework: **xUnit**. Test PDFs live in `UglyToad.PdfPig.Rendering.Skia.Tests/Documents/`.

> **Binary test assets must stay binary in git.** `.gitattributes` has `* text=auto`, but uncompressed PDFs contain no NUL bytes, so git's heuristic misdetects them as *text* and strips CR bytes on storage. A PDF's xref uses absolute byte offsets, so dropping CRs shifts the layout and breaks them: the file checks out fine on Windows (`autocrlf` restores CRLF) but corrupt on Linux/macOS (LF), giving `IndexOutOfRangeException` in `MemoryInputBytes.Seek` / `PdfTokenScanner.TryReadStream`. The bug is byte-based, not OS-based — feeding the LF blob to the renderer fails on any OS. `.gitattributes` therefore force-marks `*.pdf` (and `*.png/jpg/jpeg/gif/ico/snk`) as `binary`. After adding a new test PDF, confirm `git check-attr -a <file>` reports `binary: set`; if it was committed before the rule, run `git add --renormalize .`.

#### Image-regression tests (`PdfPigSkiaTest`)

The bulk of the suite renders a PDF page and compares it pixel-by-pixel against a committed golden PNG. Things to know before touching the renderer:

- **Must run in Release.** `PdfPigSkiaTest` `throw`s in `#if DEBUG` — image hinting/quality differs, so golden images are only valid in Release. Run `dotnet test … -c Release`. (The pure unit tests — `ParametricShadingTextureTests`, `MeshShadingDisposalTests` — run in any config.)
- **Tolerance, not exact match.** `PdfToImageHelper` allows a per-channel delta of `_threshold = 2` and up to `_maxDifferingPixelRatio = 0.001` (0.1 %) of pixels to differ, absorbing cross-platform AA/sub-pixel jitter. A failing comparison writes a diff PNG to `ErrorImages/`.
- **Golden images are committed** under `ExpectedImages/pdfpig_skia/`, with optional per-OS overrides in `ExpectedImages/{windows,linux,macos}/` (the OS-specific file wins when present, else the default is used). **Any intentional change to rendering output (e.g. tessellation, AA) means the affected goldens must be regenerated** — a green diff is not automatic.
- **Iterate fast:** add `-f net8.0` to run one TFM instead of the full matrix, and `--filter "DisplayName~<pdfname>"` to target a single document/page (e.g. `DisplayName~0000851`).

## Architecture

### Rendering Pipeline

```
PdfDocument.Open(path, SkiaRenderingParsingOptions.Instance)
  → AddSkiaPageFactory()           registers SkiaPageFactory + PageSizeFactory
  → document.GetPage<SKPicture>(n) → SkiaPageFactory.ProcessPage()
      → SkiaStreamProcessor.Process()
          → SKPictureRecorder records canvas operations
          → returns SKPicture (vector)
  → PdfPigExtensions helpers rasterize to SKBitmap / PNG
```

### Key Classes

- **`PdfPigExtensions`** — Public API surface: `AddSkiaPageFactory()`, `GetPageAsSKBitmap()`, `GetPageAsPng()`, `GetPageAsSKPicture()`, `GetPageSize()`.

- **`SkiaPageFactory`** — Implements `IPageFactory<SKPicture>`. Creates one `SkiaStreamProcessor` per page. Owns the document-scoped `SkiaFontCache`.

- **`SkiaStreamProcessor`** — Core rendering engine (internal, split across partial files by content type). Wraps a `SKPictureRecorder`; processes the PDF content stream and dispatches drawing calls.
  - `.Glyph.cs` — text/glyph rendering via HarfBuzz
  - `.Path.cs` — fill/stroke path operations
  - `.Image.cs` — image and image-mask rendering
  - `.Shading.cs` — shared shading infrastructure: the single `RenderShading` dispatch (serves both the `sh` operator and shading patterns; stroke patterns use the *stroking* alpha constant), the BBox-clip / Background / shader-draw / tile-mode helpers shared by the Axial/Radial/Function renderers, the mesh-picture cache, the bit-stream reader, the shared Type 6/7 stream-reading driver (`DrawPatchMeshUnclipped` — the record layout and edge-continuation rules are identical for Coons and Tensor), patch tessellation/texture helpers, `MapPointAffine`, and the static per-subdivision index/texcoord caches. Per-type rendering (samplers/tessellators) lives in sibling partials: `.Shading.Axial.cs`, `.Shading.Radial.cs`, `.Shading.Function.cs`, `.Shading.GouraudFree.cs` (Type 4), `.Shading.GouraudLattice.cs` (Type 5), `.Shading.Coons.cs` (Type 6), `.Shading.Tensor.cs` (Type 7). Review status and remaining known shading issues: `docs/SHADING-CODE-REVIEW.md`.
  - `.Annotations.cs` — annotation rendering

- **`PageSizeFactory`** — Lightweight `IPageFactory<PdfPageSize>` that extracts page dimensions without full rendering (handles MediaBox, CropBox, rotation, UserUnit).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BobLd/PdfPig.Rendering.Skia](https://github.com/BobLd/PdfPig.Rendering.Skia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
