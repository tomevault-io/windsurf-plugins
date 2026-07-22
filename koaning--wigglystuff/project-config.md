---
trigger: always_on
description: This repo uses a `conductor.json` setup script (`make install`) that runs
---

# Setup

This repo uses a `conductor.json` setup script (`make install`) that runs
automatically when a Conductor workspace is created. It installs all
dependencies so the environment is ready to use immediately.

# Agents

`wigglystuff` ships a small roster of AnyWidget "agents" that surface different
input modalities (sliders, speech, paint, etc.) across notebook runtimes. This
page is a quick lookup so you can see what exists and which traitlets each agent
syncs back to Python.

## Quick reference

| Agent | Module/Class | Core traitlets | One-liner |
| --- | --- | --- | --- |
| AltairWidget | `wigglystuff.altair_widget.AltairWidget` | `spec`, `width`, `height` | Flicker-free Altair chart with smooth data updates |
| ScatterLog | `wigglystuff.scatter_log.ScatterLog` | `spec`, `width`, `height` | Accumulate reactive values into a live, optionally multi-series scatter plot |
| AnnotationWidget | `wigglystuff.annotation.AnnotationWidget` | `action`, `action_timestamp`, `note`, `listening`, `actions`, `keyboard_mapping`, `gamepad_mapping`, `debounce_ms`, `width` | Annotation input surface with buttons, keyboard, gamepad, and speech-to-text |
| ApiDoc | `wigglystuff.api_doc.ApiDoc` | `doc`, `width`, `show_private` | Renders API docs for Python classes/functions |
| Slider2D | `wigglystuff.slider2d.Slider2D` | `x`, `y`, `x_bounds`, `y_bounds`, `width`, `height` | 2D pointer for coupled parameters |
| BezierCurve | `wigglystuff.bezier_curve.BezierCurve` | `points`, `samples`, `x`, `y`, `t`, `closed`, `playing`, `loop`, `interval_ms`, `duration_ms`, `sync_throttle_ms`, `show_axes`, `n_samples`, `x_bounds`, `y_bounds`, `width`, `height` | Arbitrary-degree Bezier curve editor with draggable control points, playback, and optional axis ticks |
| CurveEditor | `wigglystuff.curve_editor.CurveEditor` | `points`, `samples`, `x`, `y`, `t`, `curve`, `closed`, `playing`, `loop`, `tension`, `alpha`, `selected_index`, `show_axes`, `n_samples`, `x_bounds`, `y_bounds`, `width`, `height` | Chart-space curve editor with D3 line interpolators, path progress, and optional axis ticks |
| ChartPuck | `wigglystuff.chart_puck.ChartPuck` | `x`, `y`, `x_bounds`, `y_bounds`, `axes_pixel_bounds`, `width`, `height`, `chart_base64`, `puck_radius`, `puck_color`, `throttle` | Draggable puck overlay for matplotlib charts |
| ChartMultiSelect | `wigglystuff.chart_multi_select.ChartMultiSelect` | `selections`, `active_class`, `n_classes`, `selected_index`, `mode`, `modes`, `x_bounds`, `y_bounds`, `axes_pixel_bounds`, `width`, `height`, `chart_base64`, `selection_opacity` | Multi-region class-labeled selection on matplotlib charts |
| ChartSelect | `wigglystuff.chart_select.ChartSelect` | `mode`, `selection`, `has_selection`, `x_bounds`, `y_bounds`, `axes_pixel_bounds`, `width`, `height`, `chart_base64`, `selection_color`, `selection_opacity` | Box/lasso selection on matplotlib charts |
| Matrix | `wigglystuff.matrix.Matrix` | `matrix`, `rows`, `cols`, `min_value`, `max_value`, `step`, `mirror` | Spreadsheet-like numeric editor |
| TangleSlider | `wigglystuff.tangle.TangleSlider` | `amount`, `min_value`, `max_value`, `step`, `steps`, `pixels_per_step` | Inline slider ala Bret Victor |
| TangleChoice | `wigglystuff.tangle.TangleChoice` | `choice`, `choices` | Inline toggle among labels |
| TangleSelect | `wigglystuff.tangle.TangleSelect` | `choice`, `choices` | Dropdown version of the above |
| TangleLatex | `wigglystuff.tangle_latex.TangleLatex` | `latex`, `parameters`, `values`, `display_mode`, `editor`, `reveal_all_on_drag`, `theme`, `error` | LaTeX formula with draggable `\tangle{name}` numbers/symbols |
| SortableList | `wigglystuff.sortable_list.SortableList` | `value`, `addable`, `removable`, `editable`, `label` | Drag-and-drop ordering with optional CRUD |
| CopyToClipboard | `wigglystuff.copy_to_clipboard.CopyToClipboard` | `text_to_copy` | Copies the payload into the OS clipboard |
| ColorPicker | `wigglystuff.color_picker.ColorPicker` | `color` | Native color input with `rgb` helper |
| EdgeDraw | `wigglystuff.edge_draw.EdgeDraw` | `names`, `links`, `directed`, `width`, `height` | Sketch node/link diagrams and query adjacency |
| GridDraw | `wigglystuff.grid_draw.GridDraw` | `dots`, `lines`, `rows`, `cols`, `line_width`, `dot_radius`, `theme`, `width`, `height` | Draw dots on grid intersections and orthogonal line segments between them |
| GraphWidget | `wigglystuff.graph_widget.GraphWidget` | `nodes`, `edges`, `directed`, `width`, `height`, `selected_nodes`, `selected_edges` | Programmatic force-directed graph visualization |
| Paint | `wigglystuff.paint.Paint` | `base64`, `width`, `height`, `store_background`, `rainbow_brush`, `brush`, `marker`, `eraser`, `color_picker`, `color` | MS-Paint-style canvas with PIL helpers and a configurable toolbar |
| Excalidraw | `wigglystuff.excalidraw.Excalidraw` | `scene`, `image_base64`, `theme`, `height`, `sync_throttle_ms` | Embedded Excalidraw whiteboard (loads from CDN); `get_pil`/`save`/`from_file` helpers (`save()` remembers the path) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koaning/wigglystuff](https://github.com/koaning/wigglystuff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
