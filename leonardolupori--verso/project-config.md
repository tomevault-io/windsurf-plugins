---
trigger: always_on
description: This file provides guidance to Claude Code when working on the VERSO codebase.
---

# CLAUDE.md

This file provides guidance to Claude Code when working on the VERSO codebase.

**VERSO** — Easy Registration of SectiOns. A desktop application for registering serial histological brain sections to 3D reference atlases, replacing the QuickNII → VisuAlign → PyNutil pipeline with a single tool.

## Quick reference

```bash
uv sync                              # install all dependencies
uv run python -m verso               # launch GUI
uv run pytest                        # run all tests
uv run pytest tests/engine/          # engine tests only (no display needed)
uv run pytest tests/test_file.py::test_name  # single test
uv run ruff check src/ tests/        # lint
uv run ruff format src/ tests/       # format
```

## Architecture

### Engine / GUI separation (strict)

This is the single most important architectural rule. **Every computation lives in `engine/`. The GUI only calls engine functions and displays results.**

```
user scripts ──→ verso.engine ←── verso.gui
```

- `engine/` is pure Python. It must **never** import from PyQt6, pyqtgraph, or `verso.gui`.
- `gui/` consumes engine functions. All GUI code lives here.
- `engine/__init__.py` is the public API surface — it re-exports key functions so users can write `from verso.engine import slice_volume`.

If you are writing a function that does computation, image processing, file I/O, or data manipulation, it goes in `engine/`. If you are writing a function that creates widgets, handles mouse events, or updates the display, it goes in `gui/`.

### Package layout

```
src/verso/
├── engine/   # pure-Python computation, I/O, data model — public API in engine/__init__.py
└── gui/      # PyQt6 views, widgets, dialogs — depends on engine, never the reverse
```

### Non-destructive workflow

Original images are never modified. All preprocessing (flip, masks, contrast) is stored as parameters in `project.json` and applied on-the-fly. Copies are only created on export.

## Technology stack

- **Python 3.12**, managed by uv
- **PyQt6** — GUI framework
- **pyqtgraph** — image canvas (QGraphicsView + OpenGL viewport)
- **numpy** — array computation
- **scipy** — Delaunay triangulation (`scipy.spatial.Delaunay`)
- **opencv-python (cv2)** — image warping (`cv2.remap`)
- **tifffile** — TIFF/OME-TIFF I/O
- **scikit-image** — general image operations
- **brainglobe-atlasapi** — atlas volumes and metadata

## Key technical decisions

### Image resolution tiers

| Tier | Resolution | Purpose |
|---|---|---|
| Full resolution | Original (e.g., 20000×15000) | On disk, used only for final export |
| Working resolution | `Project.working_scale × original` | Interactive registration, masks, warping |
| Filmstrip thumbnail | ≤ `FILMSTRIP_MAX_SIDE` = 150 px on long side | Overview table, filmstrip nav |

Control points and masks are defined in working-resolution space. The ratio `Project.working_scale = working_long_side / original_long_side` is uniform across all sections — derived once at import from the largest image so its longest side fits within `THUMBNAIL_MAX_SIDE` (2000 px; see `compute_working_scale` in `engine/io/image_io.py`) — and is stored so full-resolution export can scale back up.

### Warping algorithm

**Delaunay triangulation (piecewise affine)**, matching VisuAlign's approach. Not TPS or RBF. See [.claude/warping.md](.claude/warping.md) for algorithm details and reference implementation.

Key points:
- Only the atlas overlay is warped per drag event, not the background
- The outline overlay is sampled at **display resolution** so its lines stay ~1 screen pixel wide (VisuAlign parity), capped at `_OUTLINE_MAX_SIDE` (`_OUTLINE_DRAG_MAX_SIDE` while dragging)
- The atlas slice is cached and only re-warped during a CP drag; `build_backward_remap` uses a per-triangle affine — together ~47 ms/tick (~21 fps) at the 820 px drag cap
- Four invisible corner anchors (src=dst) ensure full overlay coverage (convex hull constraint)

### QuickNII/VisuAlign compatibility

**Critical requirement.** VERSO must read and write the QuickNII/VisuAlign JSON alignment format natively. See [.claude/quint-compat.md](.claude/quint-compat.md) for format details.

### MATLAB parity for `registration.py`

**Critical requirement.** `engine/registration.py` (`VersoRegistration`) is VERSO's public entry point for non-GUI/scripting users, and has a MATLAB mirror in `matlab/+verso/VersoRegistration.m`. Whenever `registration.py`'s public API changes — a new public method, a signature change, a behavior change — the MATLAB class must be updated in the same change to keep numerical parity and a matching calling convention. See [.claude/matlab-port.md](.claude/matlab-port.md) for the axis-convention, atlas-cache, and Delaunay-warp parity notes the port depends on.

### pyqtgraph display

- Use `ImageItem` for numpy arrays (GPU-accelerated via OpenGL viewport)
- Stack two `ImageItem`s: background section (static) + atlas overlay (updated on warp)
- `pg.setConfigOption('imageAxisOrder', 'row-major')` to match NumPy convention
- Built-in histogram widget for contrast adjustment

## Coding conventions

### Python style

- Use `ruff` for linting and formatting (configured in pyproject.toml)
- Type hints on all public functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeonardoLupori/verso](https://github.com/LeonardoLupori/verso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
