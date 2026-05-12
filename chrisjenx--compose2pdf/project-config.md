---
trigger: always_on
description: **compose2pdf** is a Kotlin JVM library that renders Compose Desktop content to PDF.
---

# compose2pdf

## Project Overview

**compose2pdf** is a Kotlin JVM library that renders Compose Desktop content to PDF.

## Module Map

```
├── compose2pdf/          # Library: public API + SVG→PDF converter + font resolver
├── examples/             # Runnable examples (not published)
├── fidelity-test/        # Visual regression tests (not published)
└── docs/                 # Jekyll docs site (GitHub Pages, just-the-docs theme)
```

## Tech Stack

- **Kotlin** 2.3.20, JVM target only
- **Compose Multiplatform** 1.10.3 (Desktop)
- **Apache PDFBox** 3.0.7 (SVG→PDF conversion, image embedding, font subsetting)
- **Gradle** 8.14 — versions centralized in `gradle/libs.versions.toml`

## Build Commands

```bash
./gradlew :compose2pdf:build           # Build library
./gradlew :compose2pdf:test            # Run unit tests
./gradlew :fidelity-test:test          # Run fidelity tests (vector + raster PDF)
./gradlew :fidelity-test:test --rerun-tasks  # Force re-run (bypass Gradle cache)
./gradlew :compose2pdf:publishToMavenLocal  # Publish to ~/.m2
./gradlew :compose2pdf:compileKotlin        # Quick compile check (no tests)
./gradlew :examples:run                     # Run examples, output to examples/build/output/
open fidelity-test/build/reports/fidelity/index.html  # View fidelity report (macOS)
cd docs && bundle exec jekyll serve             # Preview docs site locally (http://localhost:4000)
```

## Fidelity Tests

Visual regression suite comparing Compose reference renders against rasterized PDFs.
- Report: `fidelity-test/build/reports/fidelity/index.html`
- Diff uses neighborhood-minimum comparison (radius=3, tolerance=8) to ignore anti-aliasing
- Gradle caches test results aggressively — use `--rerun-tasks` after changing `ImageMetrics` or `FidelityReport`

## CI

GitHub Actions (`.github/workflows/ci.yml`): build + unit tests + fidelity tests on ubuntu/macos, JDK 17. Uses `xvfb-run` for headless Compose on Linux.

Compose Multiplatform compatibility matrix (`.github/workflows/compatibility.yml`): tests against the 3 most recent CMP versions (defined in `.github/compose-versions.json`). Auto-updated weekly by `.github/workflows/update-compose-versions.yml`.

## Public API

```kotlin
renderToPdf(config, density, mode, defaultFontFamily, pagination) { content } → ByteArray  // auto-paginates by default
renderToPdf(outputStream, config, density, mode, defaultFontFamily, pagination) { content }  // streaming variant
renderToPdf(pages, config, density, mode, defaultFontFamily) { pageIndex → content } → ByteArray  // manual pages
renderToPdf(outputStream, pages, config, density, mode, defaultFontFamily) { pageIndex → content }  // streaming variant
PdfLink(href) { content }
PaginatedColumn(modifier) { content }  // page-break-aware Column, reads page config automatically
PdfRoundedCornerShape(topStart, topEnd, bottomEnd, bottomStart)
Shape.asPdfSafe()
```

Types: `PdfPageConfig` (A4/A4WithMargins/Letter/LetterWithMargins/A3/A3WithMargins + `landscape()`), `PdfMargins` (None/Narrow/Normal + `symmetric()`), `PdfPagination` (AUTO/SINGLE_PAGE), `Density`, `RenderMode` (VECTOR/RASTER), `InterFontFamily`, `Compose2PdfException`, `LocalPdfPageConfig`.

## Key Files

- `compose2pdf/src/main/kotlin/.../Compose2Pdf.kt` — Public API entry points
- `compose2pdf/src/main/kotlin/.../internal/PdfRenderer.kt` — Rendering orchestrator (vector, raster, auto-pagination)
- `compose2pdf/src/main/kotlin/.../internal/ComposeToSvg.kt` — Compose → SVG + measurement
- `compose2pdf/src/main/kotlin/.../internal/SvgToPdfConverter.kt` — SVG → PDF pages
- `compose2pdf/src/main/kotlin/.../internal/PaginatedColumn.kt` — Smart page-break layout
- `compose2pdf/src/main/kotlin/.../internal/FontResolver.kt` — Font resolution + subsetting
- `fidelity-test/src/test/.../FidelityFixtures.kt` — All fidelity test composables

## Architecture

```
Compose content → ComposeToSvg.render() → SVG string
  → SvgToPdfConverter (orchestrator)
    ├── SvgPathParser (SVG path data → PDFBox paths)
    ├── SvgColorParser + SvgColor (CSS/SVG color parsing)
    ├── SvgShapeRenderer (ellipse, rounded rect geometry)
    └── CoordinateTransform (SVG Y-down ↔ PDF Y-up)
  → PDFBox vector drawing commands → PDDocument

Callers: Compose2Pdf.kt saves PDDocument → OutputStream or ByteArray

Raster fallback: ImageComposeScene → bitmap → PDFBox embedded image

Auto-pagination: PaginatedColumn (smart page breaks)
  → ComposeToSvg.measureContentHeight() (lightweight measurement)
  → ComposeToSvg.renderWithMeasurement() (full SVG + height)
  → SvgToPdfConverter.addAutoPages() (clip + offset per page)
```

## Gotchas

### Docs Site
- **URL structure** — standalone pages (`getting-started.md`) produce `/page.html` URLs; directory index pages (`usage/index.md`) produce `/directory/` URLs. Don't use trailing slashes for standalone pages.

### Rendering
- **`@InternalComposeUiApi` opt-in required** — `CanvasLayersComposeScene` is internal Compose API
- **Variable fonts excluded** — `FontResolver.isVariableFont()` skips fonts with `fvar` table
- **SVGCanvas bezier approximation** — non-uniform rounded rects become complex bezier paths; use `PdfRoundedCornerShape`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisjenx/compose2pdf](https://github.com/chrisjenx/compose2pdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
