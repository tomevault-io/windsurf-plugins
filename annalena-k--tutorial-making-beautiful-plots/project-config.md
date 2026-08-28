---
trigger: always_on
description: > Distilled from [annalena-k/tutorial_making_beautiful_plots](https://github.com/annalena-k/tutorial_making_beautiful_plots) — a full tutorial covering the plotting workflow from data preparation to LaTeX integration, with a ready-to-use colorblind-friendly color palette package.
---

# Scientific Plotting Guide for Claude

> Distilled from [annalena-k/tutorial_making_beautiful_plots](https://github.com/annalena-k/tutorial_making_beautiful_plots) — a full tutorial covering the plotting workflow from data preparation to LaTeX integration, with a ready-to-use colorblind-friendly color palette package.

This file configures Claude's behavior when helping with scientific figure making. Follow these rules in every plotting task unless the user explicitly overrides one.

---

## Core principle

Every figure makes exactly one point. Before writing any code, identify what that point is. If you cannot state it in one sentence, ask the user before proceeding.

Remove every element that does not directly support that point. No grid lines unless axes are insufficient. No top or right spines. No redundant legend entries. No abbreviations that need caption explanations.

---

## Colors and accessibility

**Never use the default matplotlib color cycle.** Never use the `jet` colormap.

Always use a colorblind-safe palette. In order of preference:

| Use case | Palette |
|---|---|
| Default (up to 8 categories) | `okabe_and_ito()` |
| Vivid/high-contrast (up to 7) | `paul_tol_bright()` |
| Soft tones, filled areas (up to 10) | `paul_tol_muted()` |
| Paired comparisons | `nceas_two_color_pairs()` |
| Divergent data | `nceas_blue_to_red()` or `nceas_purple_to_green()` |
| 5 high-contrast colors | `ibm_design_library()` |

If the `cb_colors` package is available in the project, use it:

```python
from cb_colors import palettes
c = palettes.okabe_and_ito()
# Keys: "black", "orange", "sky_blue", "bluish_green", "yellow",
#       "blue", "vermillion", "reddish_purple"
```

If `cb_colors` is not available, paste these dicts directly into your notebook:

```python
OKABE_AND_ITO = {
    "black":          "#000000",
    "orange":         "#E69F00",
    "sky_blue":       "#56B4E9",
    "bluish_green":   "#009E73",
    "yellow":         "#F0E442",
    "blue":           "#0072B2",
    "vermillion":     "#D55E00",
    "reddish_purple": "#CC79A7",
}  # Okabe & Ito (2008) https://jfly.uni-koeln.de/color/

PAUL_TOL_BRIGHT = {
    "blue":   "#4477AA",
    "cyan":   "#66CCEE",
    "green":  "#228833",
    "yellow": "#CCBB44",
    "red":    "#EE6677",
    "purple": "#AA3377",
    "grey":   "#BBBBBB",
}  # Tol (2021) https://personal.sron.nl/~pault/data/colourschemes.pdf

PAUL_TOL_MUTED = {
    "indigo":    "#332288",
    "cyan":      "#88CCEE",
    "teal":      "#44AA99",
    "green":     "#117733",
    "olive":     "#999933",
    "sand":      "#DDCC77",
    "rose":      "#CC6677",
    "wine":      "#882255",
    "purple":    "#AA4499",
    "pale_grey": "#DDDDDD",
}  # Tol (2021) https://personal.sron.nl/~pault/data/colourschemes.pdf

IBM_DESIGN_LIBRARY = {
    "blue":    "#648FFF",
    "purple":  "#785EF0",
    "magenta": "#DC267F",
    "orange":  "#FE6100",
    "gold":    "#FFB000",
}  # IBM Design Language https://www.ibm.com/design/language/color/

ACCESSIBLE_COLORS = {
    "blue":        "#3F90DA",
    "orange":      "#FFA90E",
    "purple":      "#832DB6",
    "red":         "#BD1F01",
    "gray":        "#94A4A2",
    "dark_orange": "#E76300",
    "light_blue":  "#92DADD",
    "dark_gray":   "#717581",
    "tan":         "#B9AC70",
    "brown":       "#A96B59",
}  # Petroff (2021) https://arxiv.org/abs/2107.02270

NCEAS_TWO_COLOR_PAIRS = {
    "yellow_blue":   ["#FDB338", "#025196"],
    "tan_turquoise": ["#E3BE6B", "#3DB1A6"],
    "orange_purple": ["#EB6123", "#512888"],
    "green_purple":  ["#295E11", "#58094F"],
    "blue_red":      ["#2F67B1", "#BF2C23"],
    "blue_pink":     ["#10559A", "#DB4C77"],
    "yellow_pink":   ["#F4B301", "#DB1048"],
    "brown_blue":    ["#6A4A3C", "#0F65A1"],
}  # Phillips / NCEAS (2022), pixel-verified

NCEAS_BLUE_TO_RED = [
    "#1065AB", "#3A93C3", "#8EC4DE", "#D1E5F0", "#F9F0F9",
    "#FEDBC7", "#F6A482", "#D75F4C", "#B31529",
]  # Phillips / NCEAS (2022)

NCEAS_PURPLE_TO_GREEN = [
    "#742881", "#986EAC", "#C3A4CF", "#E5D4E8", "#F9F0F9",
    "#D9F1D5", "#ADD4A0", "#5CAE63", "#1B7939",
]  # Phillips / NCEAS (2022)
```

**Always use line style as a second visual channel** (solid, dashed, dotted) alongside color. This makes figures readable in greyscale and by colorblind readers.

---

## Figure size

Set the figure width to match the exact column width of the target journal. Include it in LaTeX without rescaling (`\includegraphics[width=\columnwidth]{...}`). This is the only way to guarantee font sizes match.

| Journal / Conference | Single column | Full text width |
|---|---|---|
| NeurIPS | 5.5 in | 5.5 in |
| ICML | 3.25 in | 6.75 in |
| PRL / PRB | 3.375 in | 6.75 in |
| MNRAS | 3.32 in | 6.97 in |
| A&A | 3.54 in | 7.28 in |

```python
columnwidth_pt = 246        # PRL single column in points (NeurIPS text width: 397.5 pt)
inches_per_pt  = 1.0 / 72.27
fig_width  = columnwidth_pt * inches_per_pt
fig_height = fig_width * 0.9  # adjust aspect ratio as needed

fig, ax = plt.subplots()
fig.set_size_inches(fig_width, fig_height)
```

---

## Style file


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [annalena-k/tutorial_making_beautiful_plots](https://github.com/annalena-k/tutorial_making_beautiful_plots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
