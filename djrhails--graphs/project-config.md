---
trigger: always_on
description: >
---


# graph-design

Economist-style data-visualisation system for matplotlib and seaborn.

> **This repo is the skill** (standard agent-skill layout: `SKILL.md` +
> [`references/`](./references) + [`examples/`](./examples)) *and* the Python
> package that powers it. Install the library with `pip install
> djrhails-graphs`; install the skill by submoduling/vendoring this repo as a
> skill directory (e.g. `skills/graph-design`). Import as `graphs`.

## Quick start

A two-series line chart with direct labels, a footnote marker, and a packed
footnote-plus-source row — the conventions the library is built for.

```python
import matplotlib.pyplot as plt
import numpy as np

from graphs import finalize, footnotes, label_lines, save_chart, set_theme, subplots

set_theme()

months = np.arange(24)
us = 2.0 + 4.0 * np.exp(-months / 9) + np.random.default_rng(0).normal(0, 0.25, 24)
eu = 1.8 + 4.5 * np.exp(-months / 11) + np.random.default_rng(7).normal(0, 0.30, 24)

fig, ax = subplots("wide")
ax.plot(months, us, label="America")
ax.plot(months, eu, label="Euro area")
label_lines(ax)

finalize(
    ax,
    title="Cooling off",
    descriptor="Headline CPI*, % change on a year earlier, monthly",
    footnote_lines=1,  # reserve room for the footnote row drawn below
)
footnotes(
    fig,
    "*All-items consumer price index",
    source="Sources: [US Bureau of Labor Statistics](https://www.bls.gov/); "
           "[Eurostat](https://ec.europa.eu/eurostat)",
)

save_chart(__file__)  # writes quick.png beside the script
```

Save to `quick.py` and run; the output sits next to it. `finalize()`
auto-sizes margins, and `footnotes()` packs the note and source onto one
row when they fit, wrapping when they don't — leave `source=` off
`finalize()` so they belong to the same line, and pass `footnote_lines=`
so auto-layout reserves the bottom band for them. `save_chart` is the
standard epilogue: tight bbox, 150 dpi, close, one-line confirmation.

### Default visual conventions

Behaviour that's automatic unless you override it:

- **Charts come in two widths.** Create figures with
  `subplots("daily")` (4.6in column, portrait-leaning) or
  `subplots("wide")` (7.0in article format) — like a newspaper's column
  formats, the width is fixed by the medium and only the height is the
  per-chart choice (`height=`). Fixed widths keep the type-to-chart
  ratio consistent across a set; ad-hoc `figsize=` widths are what make
  a gallery look ragged. Extra `plt.subplots` kwargs pass through
  (`ncols=`, `sharex=`, …).
- **Title marker is the favicon triangle** (`marker="delta"`). The hollow
  red triangle is drawn inline at the first title line's baseline, sized
  to the cap height. Pass `marker="rule"` for the legacy short red rule
  above the title, or `marker="none"` to suppress entirely.
- **Titles and descriptors auto-wrap to the figure width.** Pass them as
  single lines — `finalize()` measures with the renderer and breaks where
  *this* figure needs it (with a widow fix, so no single-word last lines).
  Never copy a reference chart's line breaks; explicit `\n` is reserved
  for semantic breaks (a descriptor's subject / unit split).
- **Explicit descriptor breaks get a semibold lead.** An explicit `\n`
  splits the descriptor into a semibold subject lead over regular
  scope/unit lines; the lead stays semibold even if it wraps. Breaks
  added by the auto-wrap alone never trigger the styling.
- **Numeric y labels sit on their gridlines** (`y_labels="on_grid"`,
  the `finalize()` default): each gridline extends into the label gutter
  and ends flush with the labels' outer edge; the label rests on its
  line, and the bottom tick inherits the dark baseline stroke. Applied
  only when the axes has visible y gridlines, so categorical axes are
  untouched; works on either side (e.g. a left latitude axis). Opt out
  with `y_labels="ticks"`, or call `y_labels_on_grid(ax)` manually on
  extra facet panels.
- **Annotations default to 9pt** — `callout`, `highlight_label`,
  `direction_label`, `threshold_arrows` match direct-label size (the
  print spec's 7.5pt reads too small at daily-chart scale).
- **Footnote markers auto-superscript.** `*, †, ‡, §, **, ††, ‡‡, §§`
  render as superscripts anywhere they appear in titles, descriptors,
  source lines, or footnote bodies — write plain text, the renderer
  handles the typography.
- **Frameless legends are default** for both `smart_legend()` and
  `top_legend()`. Boxed legends are opt-in.
- **Source + footnotes use `C_SOURCE`** (`#404040`, the styleguide's
  75% black) — darker than the muted `C_LABEL_MUTED` grey, so
  attribution stays legible while reading as metadata.
- **`finalize` always auto-layouts every margin.** It sizes all four
  `subplots_adjust` margins AND the inter-panel `wspace`/`hspace` from the
  renderer — there is no opt-out and, in the common cases, nothing to set by
  hand. Top/bottom fit the title-stack and the source/footnote band over the
  measured x-tick labels; **left/right are measured from the actual y-axis
  text** (a chart with long left-hung category labels gets a wide left, a plain
  right-axis chart keeps a tight left + a measured right); **`wspace`/`hspace`
  size a grid of panels** (wspace from the inter-column y-label width, hspace
  from the x-tick band + `panel_label` height — pass `finalize(panel_labels=True)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DJRHails/graphs](https://github.com/DJRHails/graphs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
