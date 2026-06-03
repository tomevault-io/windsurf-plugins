---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

img2ansi is a Go-based tool that converts images into ANSI art using a novel "Brown Dithering Algorithm". The project uses 2x2 pixel blocks and sophisticated Unicode character selection to create high-quality terminal art with support for multiple color palettes.

## Build Commands

```bash
# Build the main ansify command-line tool
go build ./cmd/ansify

# Build compute_tables utility (for precomputing color tables)
go build ./cmd/compute_tables

# Build compute_fonts utility (for font analysis)
go build ./cmd/compute_fonts

# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...
```

## Dependencies

- Go 1.24 or later
- `golang.org/x/image` for image processing
- For font tools: `github.com/golang/freetype`
- **Optional**: OpenCV 4 (`gocv.io/x/gocv`) - only needed for comparison tests

### Note on OpenCV

As of December 2024, the core image processing has been migrated to pure Go implementations in the `imageutil/` package. OpenCV is no longer required for normal builds. The gocv dependency is only used for comparison tests that validate the pure Go implementations against OpenCV:

```bash
# Normal build (no OpenCV required)
go build ./cmd/ansify

# Run comparison tests (requires OpenCV)
go test -tags gocv_compare ./imageutil/...
```

## The Brown Dithering Algorithm

The Brown Dithering Algorithm is a novel block-based dithering approach that converts images to ANSI art by finding the optimal representation of each 2x2 pixel block using:
- **2 colors** (foreground and background from the ANSI palette)
- **1 pattern** (one of 16 Unicode block characters)

### How It Works

For each 2x2 pixel block, the algorithm:
1. Tests all 16 Unicode block patterns (space, ▘, ▝, ▀, ▖, ▌, ▞, ▛, ▗, ▚, ▐, ▜, ▄, ▙, ▟, █)
2. For each pattern, searches for the best foreground/background color pair
3. Calculates the total error (sum of color distances for all 4 pixels)
4. Selects the pattern+colors combination with minimum error

This is essentially a constrained optimization problem: given 4 arbitrary pixel colors, find the best approximation using only 2 colors and a specific pattern.

### Example
For a 2x2 block `[red, blue, green, yellow]`, it might determine that pattern `'▛'` (three quarters) with foreground=dark_red and background=yellow gives the minimum total error.

### Perceptual Optimizations

The algorithm is specifically tuned for human perception:

1. **Edge Detection Integration**:
   - Uses Canny edge detection (thresholds 50-150) on 4x resolution intermediate image
   - Edge blocks get 50% reduced error weight (preserves sharpness)
   - Edge pixels diffuse only 50% of their error (prevents edge bleeding)
   - Results in crisp boundaries and preserved details

2. **Aspect Ratio Compensation**:
   - Default `ScaleFactor = 2.0` compensates for terminal characters being ~2x taller than wide
   - Adjustable via `-scale` flag for different terminals
   - Ensures circles remain circular, squares remain square

3. **Perceptual Color Metrics**:
   - **Redmean**: Fast approximation of human color perception
   - **LAB**: Perceptually uniform color space
   - **RGB**: Simple Euclidean (fastest but least accurate)

4. **16-Color Sweet Spot**:
   - Algorithm performs best with 16-color ANSI palette
   - Forced simplification creates stronger, more graphic shapes
   - Better pattern visibility and coherent palette
   - 256-color mode often produces "muddier" results despite higher color accuracy

## Architecture Overview

### Renderer API (v1.0+)

The library uses a `Renderer` struct that encapsulates all state for thread-safe, reusable rendering:

```go
// Create once, reuse across renders (preserves cache)
r := img2ansi.NewRenderer(
    img2ansi.WithPalette("ansi256"),
    img2ansi.WithColorMethod(img2ansi.RedmeanMethod{}),
    img2ansi.WithKdSearch(50),
)

// Render with Renderer methods
blocks := r.BrownDitherForBlocks(resized, edges)
ansi := r.RenderToAnsi(blocks)
compressed := r.CompressANSI(ansi)
```

Benefits over the old global API:
- **Thread-safe**: Multiple renderers can run concurrently
- **Cache persistence**: Block cache survives across different render sizes
- **Clean configuration**: Functional options pattern
- **Testable**: No global state to reset between tests

See `MIGRATION.md` for migrating from the old global API.

### Core Algorithm Components

1. **Image Processing** (`imageutil/` package):
   - Pure Go image processing (no OpenCV required)
   - Image loading/saving (`io.go`)
   - Resizing with various interpolation methods (`resize.go`)
   - Grayscale conversion (`convert.go`)
   - 2D convolution and sharpening (`convolve.go`)
   - Canny edge detection (`canny.go`)
   - Preprocessing pipeline (`prepare.go`):
     - `PrepareForANSI()` - all-in-one with 4x edge detection quality
     - `ResizeForANSI()` - resize only (for custom mid-pipeline processing)
     - `DetectEdges()` - edge detection only

2. **Renderer and Block Processing** (`renderer.go`, `img2ansi.go`):
   - `Renderer` struct holds palette, cache, and configuration
   - Processes images in 2x2 pixel blocks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wbrown/img2ansi](https://github.com/wbrown/img2ansi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
