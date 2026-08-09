---
trigger: always_on
description: This is a Python library for interacting with the Datawrapper API to create and manage charts.
---

# Datawrapper Python Library

This is a Python library for interacting with the Datawrapper API to create and manage charts.

## IMPORTANT: Prefer Object-Oriented API Over Legacy Methods

**For AI Agents and Developers:** This library provides two ways to interact with Datawrapper:

1. **Object-Oriented API (RECOMMENDED)** - Use chart-specific classes like `BarChart`, `LineChart`, `ColumnChart`, etc.
2. **Legacy Lower-Level API (DEPRECATED)** - Direct methods on the `Datawrapper` class like `create_chart()`, `update_chart()`, etc.

### Why Use the Object-Oriented API?

- **Type Safety**: Full type hints and IDE autocomplete support
- **Better Developer Experience**: Intuitive, Pythonic interface with method chaining
- **Rich Features**: Access to enums, validation, and chart-specific configuration options
- **Modern Design**: Follows current Python best practices and design patterns
- **Future-Proof**: The legacy API is deprecated and will be removed in a future version

### Migration Examples

**Legacy API (DEPRECATED - AVOID IN NEW CODE):**
```python
from datawrapper import Datawrapper

dw = Datawrapper(access_token="your_token")
chart_id = dw.create_chart(title="My Chart", chart_type="d3-bars")
dw.add_data(chart_id=chart_id, data=df)
dw.update_chart(chart_id=chart_id, title="Updated Title")
dw.publish_chart(chart_id=chart_id)
```

**Object-Oriented API (RECOMMENDED - USE THIS):**
```python
from datawrapper.charts import BarChart

# Method chaining approach
chart = BarChart(
    title="My Chart",
    data=df
).create().publish()

# Or step-by-step
chart = BarChart(title="My Chart", data=df)
chart.create()
chart.title = "Updated Title"
chart.update().publish()
```

### Available Chart Classes

All chart classes are in `datawrapper.charts`:
- `AreaChart` - Area charts
- `ArrowChart` - Arrow/slope charts
- `BarChart` - Horizontal bar charts
- `ColumnChart` - Vertical column charts
- `LineChart` - Line charts
- `MultipleColumnChart` - Grouped column charts
- `ScatterPlot` - Scatter plots
- `StackedBarChart` - Stacked bar charts

### Key Features of Object-Oriented API

1. **Type-Safe Enums**: Use semantic enums instead of magic strings
   ```python
   from datawrapper.charts import BarChart, NumberFormat, GridDisplay

   chart = BarChart(
       title="Sales Report",
       axis_label_format=NumberFormat.THOUSANDS_SEPARATOR,
       y_grid_display=GridDisplay.ON
   )
   ```

2. **Method Chaining**: Fluent interface for concise workflows
   ```python
   chart = BarChart(title="Sales").create().update().publish()
   ```

3. **Rich Configuration**: Chart-specific options with validation
   ```python
   from datawrapper.charts import LineChart, Line, LineWidth, LineDash

   chart = LineChart(
       title="Temperature Trends",
       lines=[
           Line(column="temp", width=LineWidth.THICK, dash=LineDash.DASHED)
       ]
   )
   ```

4. **Easy Retrieval**: Get existing charts with full type safety
   ```python
   chart = BarChart.get(chart_id="abc123")
   chart.title = "New Title"
   chart.update()
   ```

### Deprecation Warnings

The legacy API methods in `datawrapper/__main__.py` now emit `DeprecationWarning` messages:
- `create_chart()` - Use chart classes instead (e.g., `BarChart().create()`)
- `update_chart()` - Use `chart.update()` on chart instances
- `publish_chart()` - Use `chart.publish()` on chart instances
- `delete_chart()` - Use `chart.delete()` on chart instances
- `copy_chart()` - Use `chart.duplicate()` on chart instances
- `fork_chart()` - Use `chart.fork()` on chart instances
- `add_data()` - Pass data to chart constructor or set `chart.data = df`

These warnings will be visible to both human developers and AI agents, signaling that the object-oriented API should be used instead.

## Project Structure

- `datawrapper/` - Main package directory
  - `__main__.py` - Main Datawrapper API client
  - `charts/` - Chart-specific implementations
    - `base.py` - BaseChart class with common functionality
    - `models/` - Pydantic models for API metadata structures
      - `mixins.py` - Reusable mixins for shared chart functionality
      - `api_sections.py` - Models for API metadata sections
      - `text_annotations.py` - Text annotation models
      - `range_annotations.py` - Range annotation models
      - `transforms.py` - Data transformation models
    - `enums/` - Enum classes for type-safe configuration (organized by category)
      - `number_divisor.py` - NumberDivisor enum
      - `number_format.py` - NumberFormat enum
      - `date_format.py` - DateFormat enum
      - `line_width.py` - LineWidth enum
      - `line_dash.py` - LineDash enum
      - `grid_display.py` - GridDisplay enum
      - `grid_label.py` - GridLabelPosition, GridLabelAlign enums
      - `plot_height.py` - PlotHeightMode enum
      - `value_label.py` - ValueLabelDisplay, ValueLabelPlacement, ValueLabelAlignment, ValueLabelMode enums
      - `replace_flags.py` - ReplaceFlagsType enum
      - `interpolation.py` - LineInterpolation enum
      - `annos.py` - ConnectorLineType, StrokeWidth, ArrowHead enums
      - `symbol_shape.py` - SymbolShape, SymbolStyle, SymbolDisplay enums

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chekos/Datawrapper](https://github.com/chekos/Datawrapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
