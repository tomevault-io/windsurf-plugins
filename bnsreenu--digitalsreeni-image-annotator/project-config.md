---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

DigitalSreeni Image Annotator - PyQt6 desktop app for image annotation with SAM 2 integration and multi-dimensional image support.

**Fork of**: https://github.com/bnsreenu/digitalsreeni-image-annotator

## Quick Reference

```bash
# Install
pip install -e .

# Run
python -m src.digitalsreeni_image_annotator.main
# or: digitalsreeni-image-annotator
# or: sreeni
```

## Tech Stack

Python 3.10+ | PyQt6 6.7+ | Ultralytics 8.3.27 (SAM 2) | NumPy | OpenCV | Shapely

**Test suite**: `tests/` (pytest + pytest-qt). 65 tests pass on PyQt6.

## Documentation

For detailed architecture and design information, see **[docs/](docs/)**:

- **[Building Block View](docs/05_building_block_view.md)** - Components, data model, class responsibilities
- **[Runtime View](docs/06_runtime_view.md)** - Workflows and key scenarios
- **[Cross-cutting Concepts](docs/08_crosscutting_concepts.md)** - Coordinate systems, conversions, patterns
- **[Architecture Decisions](docs/09_architecture_decisions.md)** - Why we made key choices
- **[Glossary](docs/12_glossary.md)** - Terms, acronyms, data structures

See [docs/README.md](docs/README.md) for full documentation index.

## Project Structure

```
src/digitalsreeni_image_annotator/
├── main.py                    # Entry point
├── annotator_window.py        # ImageAnnotator - main window, project state
├── image_label.py             # ImageLabel - display, mouse events, rendering
├── sam_utils.py               # SAMUtils - SAM model management
├── utils.py                   # Utility functions
├── export_formats.py          # COCO, YOLO, Pascal VOC exporters
├── import_formats.py          # COCO, YOLO importers
└── [tool dialogs]             # Standalone utility windows
```

## Key Classes

| Class | File | Responsibility |
|-------|------|----------------|
| `ImageAnnotator` | annotator_window.py | Main window, state (`all_annotations`, `class_mapping`, etc.) |
| `ImageLabel` | image_label.py | Image display, zoom/pan, annotation interaction |
| `SAMUtils` | sam_utils.py | Load SAM models, run inference |

See [Building Block View](docs/05_building_block_view.md) for detailed class documentation.

## Common Development Tasks

### Adding a New Annotation Tool

1. Add button in `ImageAnnotator.create_tool_section()`
2. Set `image_label.current_tool` on click
3. Handle mouse events in `ImageLabel` (mousePressEvent, mouseMoveEvent)
4. Render in `ImageLabel.paintEvent()`
5. Call `main_window.add_annotation()` to commit

### Working with Annotations

```python
# Annotation storage: dict[image_filename, list[annotation_dict]]
self.all_annotations[self.image_file_name].append({
    "segmentation": [x1, y1, x2, y2, ...],  # Polygon
    # OR "bbox": [x, y, width, height],     # Rectangle
    "category": class_name
})
```

### SAM Integration

SAM runs in-process; the Ultralytics model object lives on `SAMUtils`
and persists across calls. Inference runs on a background QThread but
the public API is synchronous — see ADR-013 in
`docs/09_architecture_decisions.md`.

```python
# Load model on first selection (downloads weights if missing, ~40-400MB)
self.sam_utils.change_sam_model("SAM 2 tiny")  # blocks UI thread via QEventLoop spin

# Run inference (also runs on worker thread, returns when done)
prediction = self.sam_utils.apply_sam_points(
    qimage,
    positive_points=[(x1, y1)],
    negative_points=[(x2, y2)]
)
# Returns: {"segmentation": [...], "score": float}
```

## Important Notes

### Platform Support
- ✅ Windows, macOS, Linux supported (PyQt6 native integration improved over PyQt5)
- Linux runtime needs libxcb-cursor0 (Qt6 requires this; was optional under Qt5)

### Critical: Project Loading
**Always check `is_loading_project` flag before saving!** Autosave during load corrupts files (v0.8.12 fix).

```python
def save_project(self):
    if self.is_loading_project:
        return  # Skip during load
    # ... save logic
```

### Coordinate Systems
- **Mouse events**: Screen coordinates → must convert to image coordinates
- **Annotations stored**: Image coordinates (unzoomed, absolute pixels)
- Account for `zoom_factor`, `offset_x`, `offset_y`

See [Cross-cutting Concepts](docs/08_crosscutting_concepts.md#coordinate-systems) for details.

### Multi-dimensional Images
- User assigns dimensions (T, Z, C, H, W) via dialog
- Slices extracted with names like `stack.tif_T0_Z5_C0`
- Each slice annotated independently
- Stored in `image_slices` dict
- TIFF axis hint: `load_tiff` reads `tifffile.series[0].axes` and pre-fills the dimension dialog; ndim≥5 had a `[-ndim:]` slice bug that produced 2560 wrong slices on a 5D `TZCYX` file — see arc42 if you touch this

See [Runtime View](docs/06_runtime_view.md#multi-dimensional-image-loading) for workflow.

### Patterns introduced in v0.9.0 (read before touching these areas)

| Area | Pattern | Why |
|------|---------|-----|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bnsreenu/digitalsreeni-image-annotator](https://github.com/bnsreenu/digitalsreeni-image-annotator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
