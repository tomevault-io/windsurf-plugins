---
trigger: always_on
description: PlotJS is a Python package that transforms static matplotlib charts into interactive web visualizations. It exports matplotlib figures as SVG, parses them with JavaScript, and adds browser-based interactivity (tooltips, hover effects, grouping) without requiring chart re-serialization.
---

# PlotJS - LLM Reference Guide

## Project Overview

PlotJS is a Python package that transforms static matplotlib charts into interactive web visualizations. It exports matplotlib figures as SVG, parses them with JavaScript, and adds browser-based interactivity (tooltips, hover effects, grouping) without requiring chart re-serialization.

**Core Philosophy:** Leverage matplotlib's native SVG output + JavaScript DOM manipulation instead of recreating charts in D3/Altair.

## Quick Architecture

```
Matplotlib Figure → SVG Export (Python) → HTML Template (Jinja2) → Interactive Browser
```

1. **Python (PlotJS class):** Captures matplotlib figure as SVG string, collects tooltip/styling metadata
2. **Jinja2 Template:** Injects SVG + CSS + JavaScript parser + configuration into HTML
3. **Browser (PlotSVGParser):** Parses SVG structure to identify plot elements, attaches hover interactivity

## How to run command

Always use `uv` and/or `just` for running commands:

- `uv run pytest tests/test-python`
- `uv run pytest tests/test-python`
- `uv run python -c "import matplotlib"`

## Key Components

### Python Module (`/plotjs/`)

**`plotjs.py`** - Core `PlotJS` class with method chaining

- `__init__(fig, **savefig_kws)` - Converts matplotlib figure to SVG
- `add_tooltip(labels, groups, hover_nearest, ax)` - Configure hover tooltips
- `add_css(from_string)` - Add custom CSS styling
- `add_javascript(from_string)` - Add custom JavaScript
- `save(file_path)` / `as_html()` - Export to file or return HTML string
- Internal: `_set_plot_data_json()`, `_set_html()` - Prepare template data

**`css.py`** - CSS utilities

- `from_dict(css_dict)` - Convert Python dict to CSS
- `from_file(css_file)` - Load external CSS
- `is_css_like(s)` - Validate CSS syntax

**`javascript.py`** - JavaScript utilities

- `from_file(javascript_file)` - Load external JavaScript

**`utils.py`** - Internal helpers

- `_vector_to_list(vector, name)` - Convert pandas/numpy/lists using Narwhals
- `_get_and_sanitize_js(file_path, after_pattern)` - Extract JS code

**`data/datasets.py`** - Sample datasets (iris, mtcars, titanic) with Narwhals support

### Static Assets (`/plotjs/static/`)

**`template.html`** - Jinja2 template structure

- Injects: `{{ svg }}`, `{{ default_css }}`, `{{ additional_css }}`, `{{ js_parser }}`, `{{ plot_data_json }}`
- Creates tooltip container and event handling

**`plotparser.js`** - `PlotSVGParser` JavaScript class

- `findBars(svg, axes_class)` - Select bar chart elements
- `findPoints(svg, axes_class, tooltip_groups)` - Select scatter points
- `findLines(svg, axes_class)` - Select line chart elements
- `findPies(svg, axes_class)` - Select pie chart elements
- `findAreas(svg, axes_class)` - Select filled area elements
- `nearestElementFromMouse(mouseX, mouseY, elements)` - Hover nearest detection
- `setHoverEffect(...)` - Attach mouseover handlers, show tooltips

**`default.css`** - Base styling for tooltips and hover states

## Technical Implementation Details

### SVG Parsing Strategy

Challenge: Identify plot elements in SVG without metadata.

Solution: Pattern-based CSS selectors targeting matplotlib's SVG structure:

| Element        | SVG Pattern                            | Selector                                                 |
| -------------- | -------------------------------------- | -------------------------------------------------------- |
| Scatter Points | `<g id="PathCollection_N"> <use>`      | `g#axes_class g[id^="PathCollection"] use`               |
| Lines          | `<g id="line2d_N"> <path>`             | `g#axes_class g[id^="line2d"] path` (exclude axis lines) |
| Bars           | `<g id="patch_N"> <path>`              | `g#axes_class g[id^="patch"] path[clip-path]`            |
| Areas          | `<g id="FillBetweenPolyCollection_N">` | `g#axes_class g[id^="FillBetweenPolyCollection"] path`   |

### Data Flow: Python → JavaScript

1. Python collects configuration:

```python
plot_data_json = {
    "tooltip_labels": [...],
    "tooltip_groups": [...],
    "tooltip_x_shift": 10,
    "tooltip_y_shift": -10,
    "hover_nearest": False,
    "axes": {"axes_1": {...}, "axes_2": {...}}
}
```

2. Jinja2 injects as JSON in HTML template

3. JavaScript accesses via `plot_data["axes"]["axes_1"]["tooltip_labels"]`

### Method Chaining

All methods return `self` for fluent API:

```python
PlotJS(fig).add_tooltip(...).add_css(...).save(...)
```

### Multiple Axes Support

Each axes processed independently via `ax` parameter:

```python
PlotJS(fig).add_tooltip(labels1, ax=ax1).add_tooltip(labels2, ax=ax2).save(...)
```

### Reproducibility

Optional `seed` parameter ensures deterministic UUID generation for consistent output.

## File Structure

```
plotjs/
├── __init__.py              # Package exports
├── plotjs.py                  # Core PlotJS class
├── css.py                   # CSS utilities
├── javascript.py            # JavaScript utilities
├── utils.py                 # Internal helpers
├── data/
│   ├── datasets.py          # Sample datasets with Narwhals

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [y-sunflower/plotjs](https://github.com/y-sunflower/plotjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
