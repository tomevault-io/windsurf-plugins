---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "cute-viz", a Python package for visualizing CuTe tensor layouts from NVIDIA's CUTLASS library. The package provides functions to render CuTe layouts as SVG images for better understanding of GPU tensor memory layouts and thread mappings.

## Development Commands

The project uses a standard Python package structure with pyproject.toml configuration:

- **Install in development mode**: `pip install -e .`
- **Install from GitHub**: `pip install git+https://github.com/NTT123/cute-viz.git`
- **Run examples**: `python examples/layout_example.py` or `python examples/mma_atom_example.py`

## Project Structure

- `cute_viz/` - Main package directory
  - `__init__.py` - Package initialization, exports public API:
    - Visualization: `render_layout_svg`, `render_tv_layout_svg`, `render_swizzle_layout_svg`, `render_copy_layout_svg`, `render_tiled_copy_svg`, `render_mma_layout_svg`, `render_tiled_mma_svg`
    - Display: `display_svg`, `display_layout`, `display_tv_layout`, `display_swizzle_layout`, `display_copy_layout`, `display_tiled_copy`, `display_mma_layout`, `display_tiled_mma`
    - Utilities: `tidfrg_S`, `tidfrg_D`
  - `core.py` - Core visualization functions with internal helpers and utility functions
- `examples/` - Runnable example scripts demonstrating package usage
- `pyproject.toml` - Project configuration with dependencies
- `README.md` - Package documentation with usage examples
- `.python-version` - Python version specification (>=3.12)

## Dependencies

The package requires:
- numpy - Array operations
- nvidia-cutlass-dsl>=4.2.0 - CUTLASS CuTe library
- cuda-bindings>=12.9.1 - CUDA bindings
- cuda-python>=12.9.1 - CUDA Python support
- svgwrite - SVG generation
- ipython - Jupyter notebook display support

## Architecture Notes

The package provides four types of layout visualizations:

1. **Basic Layout Visualization** (`render_layout_svg`, `display_layout`):
   - Visualizes CuTe layouts as color-coded grids with grayscale colors
   - Shows the linear index mapping for 2D tensor coordinates
   - Cell size: 20px, supports up to 8 distinct grayscale shades for different indices

2. **Thread-Value (TV) Layout Visualization** (`render_tv_layout_svg`, `display_tv_layout`):
   - Visualizes rank-2 TV layouts showing thread-to-memory mappings
   - Requires both a layout and a tile_mn parameter (rank-2 MN Tile)
   - Uses composition with identity tensor to map coordinates
   - Color-codes threads with 8 distinct pastel colors
   - Labels each cell with thread ID (T#) and value ID (V#)

3. **Swizzle Layout Visualization** (`render_swizzle_layout_svg`, `display_swizzle_layout`):
   - Visualizes swizzled layouts that permute elements for better memory access patterns
   - Uses the same grayscale color scheme as basic layouts
   - Internally attempts to convert to position-independent form if available
   - Shows the permuted memory access pattern created by the swizzle transformation

4. **Copy Layout Visualization** (`render_copy_layout_svg`, `display_copy_layout`):
   - Visualizes copy operations with source and destination TV layouts side-by-side
   - Shows how threads map to both source and destination memory locations
   - Two grids displayed horizontally with 3-cell gap between them
   - Uses same pastel color scheme as TV layouts (8 colors by thread ID)
   - Labels cells with thread ID (T#) and value ID (V#)
   - Useful for understanding data movement patterns in GPU memory copies

All visualization functions use SVG format with:
- 20px cell size (hardcoded)
- Black cell borders
- 8px font size for labels
- Color cycling based on modulo of index/thread count

The package supports both file-based output (render functions) and direct Jupyter notebook display (display functions). All CuTe operations must be wrapped in `@cute.jit` decorator.

## Python CuTe DSL Critical Notes

**IMPORTANT: Python CuTe DSL differs significantly from C++ CuTe API.**

### Creating Swizzle Layouts

**❌ WRONG - Don't use class constructors directly:**
```python
swizzle = cute.Swizzle(2, 3, 3)  # TypeError! Constructor expects MLIR IR Value
```

**✅ CORRECT - Use factory functions:**
```python
swizzle = cute.make_swizzle(b=2, m=3, s=3)  # Factory function creates proper MLIR representation
```

### Composing Swizzles with Layouts

**❌ WRONG - Using composition() hangs/fails:**
```python
swizzled_layout = cute.composition(base_layout, swizzle)  # Will hang indefinitely!
```

**✅ CORRECT - Use make_composed_layout:**
```python
# Creates composition: swizzle ∘ offset ∘ base_layout
swizzled_layout = cute.make_composed_layout(swizzle, 0, base_layout)
```

**Key Differences:**
- `composition(lhs, rhs)`: General-purpose layout-to-layout composition, computes `lhs(rhs(c))`
- `make_composed_layout(inner, offset, outer)`: Specifically designed for transformations like swizzles, creates `inner ∘ offset ∘ outer`
- **For swizzles, you MUST use `make_composed_layout()`**, not `composition()`

### Swizzle Parameters

Swizzles are defined by three parameters (b, m, s):
- **b (BBits)**: Number of bits in the mask

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NTT123/cute-viz](https://github.com/NTT123/cute-viz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
