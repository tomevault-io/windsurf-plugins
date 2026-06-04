---
trigger: always_on
description: This package is a Python implementation of a ggplot2-like grammar that renders
---

# AGENTS.md - Project Context for AI Assistants

## Codex Working Notes

This package is a Python implementation of a ggplot2-like grammar that renders
to Plotly. Treat ggplot2/R compatibility as the default product requirement:
function names, argument names, aliases, and behavior should match ggplot2
unless Python makes that impossible. Do not rename public parameters to more
Pythonic spellings when a ggplot2 spelling already exists; preserve aliases such
as `colour`, `linewidth`, `show.legend`, `na.rm`, and `position_*`.

### Rendering Architecture

- `ggplotly/ggplot.py` owns composition and final draw ordering: layers, scales,
  coords, theme, labels, guides, annotations, and size.
- Most geoms convert mapped data into Plotly traces through `Geom._transform_fig`
  and `ggplotly/trace_builders.py`.
- `ggplotly/aesthetic_mapper.py` is the shared resolver for `color`, `fill`,
  `size`, `shape`, `linetype`, `group`, and alpha.
- Keep computation-heavy or invariant-heavy rendering logic in small pure helper
  functions that return values/specs; keep Plotly `fig.add_trace` and
  `fig.update_layout` mutation inside geom/theme/layout shells.
- Avoid broad manager/service abstractions. Add a helper module only when it
  encodes a real rendering concept or is reused/tested directly.

### Important Semantics Learned During Visual Review

- `fill` and `color` have distinct ggplot2 meanings. For bars/histograms,
  `fill` is interior color and `color`/`colour` is outline color.
- Do not change the shared `position_dodge` math casually. Different geoms may
  use it differently; bar/column chart dodging can be implemented at the
  trace/layout level without changing the position object itself.
- `geom_bar()` defaults to stacked bars when `fill` is mapped.
  `position="dodge"` / `position_dodge()` should render side-by-side groups.
  `position_fill()` should normalize each x stack to 1.0.
- `geom_density(fill=...)` should fill the area under the density curve, not
  only style the line.
- Map overlays must use geo traces when a geo context exists. Cartesian
  `Scatter`/`Heatmap` overlays detach visually from Plotly geo projections.
- Date/time index values must stay as datetimes in rendered traces; avoid
  accidentally converting `DatetimeIndex` values to nanosecond integers.
- Mapped numeric point sizes should be scaled to a visible marker range, not
  passed through raw data values.

### Testing And Verification

- Preferred full test command:
  ```bash
  .venv/bin/python -m pytest pytest -q --no-cov
  ```
- Add targeted tests near the behavior owner, and use
  `pytest/test_fp_review_regressions.py` for cross-cutting rendering regressions
  found during review.
- For notebook visual review, execution passing is not enough. Render notebooks
  to HTML, capture PNGs, and inspect whether the chart semantics are correct.
- Use npm/Node Playwright with its managed Chromium browser for notebook PNG
  capture. Do not use Kaleido or the installed macOS Chrome app for bulk review;
  those paths can launch the user's desktop Chrome profile/updater and cause
  hangs or crashes.
- In this environment, Jupyter notebook execution may need permission to bind
  local kernel ports. If nbconvert fails with `PermissionError` while finding a
  port, rerun the same command with escalation rather than changing notebook
  code.

### Git Hygiene

- Work on feature branches and commit coherent chunks frequently.
- Do not rewrite or revert unrelated user changes. If the working tree is dirty,
  inspect and preserve user-owned changes.
- Keep generated review artifacts under `/private/tmp` unless the user asks for
  repo-tracked artifacts.

## Development Philosophy

**IMPORTANT**: This library aims to faithfully replicate R's ggplot2 API in Python.

When contributing or modifying code:
1. **Follow ggplot2 conventions** - Match R's ggplot2 function names, parameter names, and behavior as closely as possible
2. **Consult ggplot2 documentation** - When implementing existing ggplot2 features, reference https://ggplot2.tidyverse.org/reference/
3. **Extrapolate for new features** - For functionality not in ggplot2 (e.g., `geom_candlestick`, `geom_stl`, `geom_sankey`), follow ggplot2 naming conventions and design patterns:
   - Use `geom_*` prefix for geometric objects
   - Use `stat_*` prefix for statistical transformations
   - Use `scale_*_*` pattern for scales (e.g., `scale_x_log10`, `scale_color_manual`)
   - Accept `aes()` mappings consistently
   - Support `data=` parameter override in geoms
4. **Pythonic adaptations** - Only deviate from ggplot2 when Python requires it (e.g., strings for column names in `aes()`)

## Project Overview

**GGPLOTLY** is a Python data visualization library that combines R's ggplot2 Grammar of Graphics with Plotly's interactive capabilities.

- Version: 0.3.5 (Beta)
- Author: Ben Cho
- Python: 3.9+
- License: MIT

## Quick Start

```python
from ggplotly import ggplot, aes, geom_point, theme_minimal

(ggplot(df, aes(x='col1', y='col2', color='category'))
 + geom_point()
 + theme_minimal())
```

## Project Structure

```
ggplotly/
├── ggplotly/           # Main package
│   ├── ggplot.py       # Core ggplot class
│   ├── aes.py          # Aesthetic mappings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbcho/ggplotly](https://github.com/bbcho/ggplotly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
