---
trigger: always_on
description: from charted import (
---

# AGENTS.md: Charted (Python SVG Chart Library)

## Quick API Reference

```python
import charted
from charted import (
    BarChart, ColumnChart, LineChart, ScatterChart,
    PieChart, RadarChart, AreaChart, BoxPlot, Histogram,
    HeatmapChart, GanttChart,
)

# Bar (horizontal)
BarChart(data=[10, 20, 30], labels=["A", "B", "C"], title="Sales")

# Column (vertical)
ColumnChart(data=[10, 20, 30], labels=["A", "B", "C"])

# Line
LineChart(data=[[10, 20, 30]], labels=["Jan", "Feb", "Mar"])

# Scatter (note: uses x_data/y_data, NOT data=)
ScatterChart(x_data=[1, 2, 3], y_data=[10, 20, 30])

# Pie
PieChart(data=[35, 28, 18], labels=["A", "B", "C"])

# Radar
RadarChart(data=[85, 90, 75, 88, 92], labels=["Spd", "Str", "Def", "Tech", "Sta"])

# Area
AreaChart(data=[10, 20, 15, 25], labels=["Q1", "Q2", "Q3", "Q4"])

# Box Plot: each item in data is a raw distribution list
BoxPlot(data=[[1,2,3,4,5,6,7], [2,4,6,8,10]], labels=["A", "B"])

# Histogram: single flat list, bins param
Histogram(data=[1.2, 2.3, 2.5, 3.1, 4.0], bins=5)

# Heatmap: 2D matrix
HeatmapChart(data=[[1,2,3],[4,5,6],[7,8,9]], labels=["R1","R2","R3"])

# Gantt
GanttChart(tasks=[{"name": "Task 1", "start": 0, "end": 5}])
```

## Common Patterns

### Single series
```python
chart = BarChart(data=[120, 180, 210], labels=["Q1", "Q2", "Q3"], title="Revenue")
chart.save("chart.svg")
```

### Multi-series
```python
chart = ColumnChart(
    data=[[12, 22, 30], [-8, -15, -20]],
    labels=["Q1", "Q2", "Q3"],
    series_names=["Revenue", "Costs"],
)
```

### Stacked vs side-by-side
```python
# Column: stacked by default (y_stacked=True). Disable:
ColumnChart(data=[[1,2],[3,4]], labels=["A","B"], y_stacked=False)

# Bar: side-by-side by default. Stack with:
BarChart(data=[[1,2],[3,4]], labels=["A","B"], x_stacked=True)
```

### Dark theme
```python
chart = BarChart(data=[1,2,3], labels=["A","B","C"], theme="dark")
```

### Custom dimensions
```python
chart = LineChart(data=[[1,2,3]], labels=["A","B","C"], width=800, height=400)
```

## Auto Chart Type

```python
from charted import auto

chart = auto([10, 20, 30])                    # <= 6 items -> PieChart
chart = auto([10, 20, 30, 40, 50, 60, 70])   # > 6 items -> BarChart
chart = auto([[1,2,3,4,5],[6,7,8,9,10]])      # few rows, many cols -> ColumnChart
chart = auto({"col_a": [1,2,3], "col_b": [4,5,6]})  # dict -> from_dataframe
```

## Output Formats

```python
chart = BarChart(data=[1,2,3], labels=["A","B","C"])

chart.save("out.svg")              # SVG file
chart.save("out.png")              # PNG (requires cairosvg)
chart.save("out.png", scale=3)     # PNG at 3x resolution

svg_str = chart.to_svg()           # Raw SVG string
html_str = chart.to_html()         # Standalone HTML with embedded SVG
b64_uri = chart.to_base64()        # data:image/svg+xml,... URI
md_str = chart.to_markdown()       # Markdown image tag with inline data URL
```

## Themes

### Built-in presets
```python
chart = BarChart(data=[1,2,3], labels=["A","B","C"], theme="light")     # default
chart = BarChart(data=[1,2,3], labels=["A","B","C"], theme="dark")
chart = BarChart(data=[1,2,3], labels=["A","B","C"], theme="high-contrast")
```

### Register custom theme
```python
from charted import Theme, register_theme

register_theme("corporate", Theme(
    background_color="#1a1a2e",
    text_color="#eaeaea",
    colors=["#0f3460", "#e94560", "#16213e"],
))
chart = BarChart(data=[1,2,3], labels=["A","B","C"], theme="corporate")
```

### Fluent style override
```python
chart = BarChart(data=[1,2,3], labels=["A","B","C"]).style(
    background_color="#000", text_color="#fff", font_size=14
)
```

### Named palettes
```python
from charted.themes.core import NAMED_PALETTES, resolve_palette
# Available: default, viridis, ocean, categorical, rainbow, monochrome,
#            pastel, sunset, forest, inferno
colors = resolve_palette("viridis")
```

## Data Loading

```python
from charted import load_csv, load_json, load_data

# Generic (auto-detects .csv/.json/.tsv)
x, y, labels = load_data("sales.csv", x_col="Quarter", y_col="Revenue")

# Specific
x, y, labels = load_csv("sales.csv", x_col="Quarter", y_col="Revenue")
x, y, labels = load_json("data.json")
```

### From pandas DataFrame
```python
from charted import from_dataframe
chart = from_dataframe(df, chart_type="BarChart", title="Sales")
```

### From dict
```python
from charted import from_dict
chart = from_dict({
    "chart_type": "BarChart",
    "data": [10, 20, 30],
    "title": "Sales",
})
```

## Config Serialization

```python
# Save chart config for later replay
config = chart.to_config()  # -> dict with chart_type, data, labels, theme, etc.

# Recreate chart from config
from charted.charts.chart import Chart
new_chart = Chart.from_config(config)

# Override specific params on recreation
new_chart = Chart.from_config(config, title="Updated Title", width=800)
```

## describe()

Returns structured metadata for agent reasoning about a chart:

```python
chart = BarChart(data=[120, -50, 210], labels=["Q1", "Q2", "Q3"], title="P&L")
info = chart.describe()
# {
#   "chart_type": "BarChart",
#   "title": "P&L",
#   "dimensions": {"width": 500, "height": 500},
#   "series": [{"name": None, "count": 3, "min": -50.0, "max": 210.0, "mean": 93.33, "sum": 280.0}],
#   "labels": ["Q1", "Q2", "Q3"],
#   "label_count": 3,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marzukia/charted](https://github.com/marzukia/charted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
