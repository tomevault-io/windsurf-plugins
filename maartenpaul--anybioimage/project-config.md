---
trigger: always_on
description: Interactive anywidget for visualizing biological images in Jupyter and marimo notebooks.
---

# anybioimage

Interactive anywidget for visualizing biological images in Jupyter and marimo notebooks.

## Features

- **Multi-dimensional support** - 5D images (TCZYX: Time, Channel, Z-stack, Y, X)
- **Multi-layer mask overlays** - Customizable colors, opacity, and contour rendering
- **Annotation tools** - Rectangles, polygons, and points with visibility controls
- **SAM integration** - Segment Anything Model for interactive segmentation
- **Tile-based rendering** - Viewport-aware caching: precomputes visible tiles first across all T/Z, then fills the rest off-screen
- **BioImage format support** - TIFF, OME-Zarr, and other formats via bioio
- **HCS plate support** - OME-Zarr plate visualization with well and FOV selection

## Project Structure

```
anybioimage/
├── __init__.py           # Public API exports
├── utils.py              # Image processing helpers and color constants
├── viewer.py             # Main BioImageViewer widget (JS/CSS frontend + traitlets)
└── mixins/
    ├── __init__.py       # Mixin exports
    ├── image_loading.py  # Image loading, caching, and prefetching
    ├── mask_management.py # Mask layer operations
    ├── annotations.py    # Annotation data management (ROIs, polygons, points)
    ├── plate_loading.py  # HCS OME-Zarr plate loading (well/FOV selection)
    └── sam_integration.py # SAM model integration
```

## Development

This project uses `uv` for package management:

```bash
uv pip install -e ".[all]"      # Recommended: all dependencies except SAM
uv pip install -e ".[dev]"      # Development dependencies only
uv pip install -e ".[sam]"      # SAM model support (requires PyTorch)
uv pip install -e ".[complete]" # Everything including SAM
```

**Note:** The `sam` extra requires PyTorch and may not work on Python 3.13+. Use Python 3.10-3.12 for SAM features.

## Usage

```python
from anybioimage import BioImageViewer
from bioio import BioImage

# Load and display an image
viewer = BioImageViewer()
img = BioImage("image.tif")
viewer.set_image(img)
viewer
```

### HCS Plate Usage

```python
from anybioimage import BioImageViewer

viewer = BioImageViewer()
viewer.set_plate("plate.zarr")  # OME-Zarr HCS plate
viewer
```

The widget adds **Well** and **FOV** dropdown selectors. Changing the well loads its FOV list; changing the FOV loads the corresponding image with full T/Z/channel support.

**Plate traitlets:** `plate_wells`, `plate_fovs`, `current_well`, `current_fov`

## Key Classes

### BioImageViewer

The main widget class with these capabilities:

- `set_image(data)` - Load numpy array or BioImage object
- `set_plate(path)` - Load HCS OME-Zarr plate with well/FOV selection dropdowns
- `add_mask(labels, name, color, opacity)` - Add mask overlay layer
- `enable_sam(model_type)` - Enable SAM segmentation
- `rois_df`, `polygons_df`, `points_df` - Access annotation data as DataFrames

### Annotation Tools

- **Pan** - Navigate the image
- **Select** - Select existing annotations
- **Rectangle** - Draw bounding boxes (triggers SAM when enabled)
- **Polygon** - Draw polygon regions
- **Point** - Place point markers (triggers SAM when enabled)

## Marimo Integration

When editing marimo notebooks, only edit contents inside `@app.cell` decorators:

```python
@app.cell
def _():
    viewer = BioImageViewer()
    viewer.set_image(image_data)
    mo.ui.anywidget(viewer)
    return
```

Run `marimo check --fix` after editing to catch formatting issues.

## Testing with Playwright

Use the `playwright-cli` skill to test the widget in a real browser against a running marimo server.

### Screenshots

Store all Playwright screenshots in a temporary folder at `/tmp/anybioimage-screenshots/`. Create it at the start of a testing session and delete it when done:

```bash
mkdir -p /tmp/anybioimage-screenshots
# ... run tests, save screenshots to /tmp/anybioimage-screenshots/
rm -rf /tmp/anybioimage-screenshots
```

In playwright-cli, pass the temp path when taking screenshots:
```javascript
await page.screenshot({ path: '/tmp/anybioimage-screenshots/step-name.png' });
```

### Setup

```bash
# Start marimo server (note the access token in the URL printed to terminal)
marimo edit examples/image_notebook.py

# Open browser (use chromium, not chrome — chrome requires root to install)
playwright-cli open "http://localhost:2718?access_token=<token>" --browser=chromium
```

### Key patterns for testing anybioimage widgets

**The widget renders inside a shadow DOM** (`MARIMO-ANYWIDGET` element). Regular DOM queries won't find the canvas — use `element.shadowRoot`:

```javascript
// Find the canvas
for (const el of document.querySelectorAll('*')) {
  if (el.tagName === 'MARIMO-ANYWIDGET' && el.shadowRoot) {
    const canvas = el.shadowRoot.querySelector('canvas');
    // canvas is here
  }
}
```

**The widget output is in a scrollable container**, not `window`. Scroll it directly:

```javascript
// Scroll to widget
let parent = widgetElement.parentElement;
while (parent) {
  if (parent.scrollHeight > 2000 && parent.clientHeight < 1000) {
    parent.scrollTo(0, targetY);
    break;
  }
  parent = parent.parentElement;
}
```

**Sliders in the widget** (in order):
- Index 0: Brightness (min=-1, max=1)
- Index 1: Contrast (min=-1, max=1)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maartenpaul/anybioimage](https://github.com/maartenpaul/anybioimage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
