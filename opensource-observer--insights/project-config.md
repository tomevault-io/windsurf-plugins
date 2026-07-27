---
trigger: always_on
description: This guide covers converting Hex notebooks (exported as `.ipynb` files from Hex.tech) to marimo Python notebooks, maintaining functionality while adhering to marimo's reactive programming model.
---

# Converting Hex Notebooks to marimo Notebooks

## Overview
This guide covers converting Hex notebooks (exported as `.ipynb` files from Hex.tech) to marimo Python notebooks, maintaining functionality while adhering to marimo's reactive programming model.

Both Hex and marimo support a mix of SQL, markdown, Python, and interactive UI elements. The key difference lies in their execution model:

**Hex (Traditional Notebook Model):**
- Cells execute top-to-bottom in order
- Can have hidden state (like Jupyter notebooks)
- Variables can be reassigned/shadowed across cells
- Manual cell execution control

**marimo (Reactive Notebook Model):**
- Automatic reactivity: cells re-run when dependencies change
- No hidden state: all dependencies must be explicit
- Each cell must return unique variable names (no shadowing)
- Data flows through explicit dependency declarations

**Charting:**
Both Hex and marimo support Altair and Plotly. Since Hex uses Altair by default, it's easiest to continue using Altair when converting to marimo.

## Core Principles

### 1. Notebook Structure Pattern
A typical marimo notebook follows this organization:
1. **Title cell**: Heading and metadata using `mo.md()`
2. **Context/Introduction**: Overview, key findings, data sources
3. **Configuration**: UI controls and constants
4. **Data loading**: Database queries, file reads
5. **Data processing**: Transformations and aggregations
6. **Analysis & Visualizations**: Insights with charts/tables
7. **Helper functions**: Reusable utilities
8. **Setup cell**: Environment and database initialization (always last)

### 2. Cell Organization Pattern
```python
# Standard cell structure
@app.cell(hide_code=True)  # Use hide_code for presentation/output cells
def _(dependency1, dependency2, mo, pd):
    # Cell logic here
    result = process_data(dependency1, dependency2)
    return (result,)  # ALWAYS return tuple, even for single values
```

### 3. Key Differences from Hex
- **Reactive execution**: Cells automatically re-run when dependencies change
- **No hidden state**: All dependencies must be explicitly declared in function signatures
- **No variable shadowing**: Each cell must return unique variable names (can't reassign variables across cells)
- **No circular references**: Data must flow in one direction
- **Pure Python files**: marimo notebooks are `.py` files, not `.ipynb` (better for version control)

## Data Loading Conversion

### SQL Queries
**Hex pattern:**
```python
# Hex uses direct SQL with automatic variable interpolation
df = sql("SELECT * FROM table WHERE date > '{start_date}'")
```

**marimo pattern:**
```python
@app.cell(hide_code=True)
def _(mo, pyoso_db_conn, START_DATE, END_DATE):
    _query = f"""
    SELECT 
        column1,
        column2,
        column3
    FROM table_name
    WHERE date BETWEEN '{START_DATE}' AND '{END_DATE}'
    ORDER BY date
    """
    df_result = mo.sql(_query, engine=pyoso_db_conn, output=False)
    return (df_result,)
```

**Notes:**
- Use `mo.sql()` with `output=False` to prevent automatic display
- Reference configuration variables (like `START_DATE`) in function signature
- Use f-strings for variable interpolation
- `pyoso_db_conn` is provided by the `setup_pyoso()` cell (always last)

### File-Based Data
**Hex pattern:**
```python
# Hex Google Sheets integration
df = sql("SELECT * FROM hex.gsheets.sheet_name")
```

**marimo pattern:**
```python
@app.cell(hide_code=True)
def _(pd):
    from pathlib import Path
    
    # CSV files
    df = pd.read_csv('path/to/data.csv')
    
    # Or Parquet for larger datasets
    # df = pd.read_parquet('path/to/data.parquet')
    
    # Or Excel
    # df = pd.read_excel('path/to/data.xlsx', sheet_name='Sheet1')
    
    return (df,)
```

## UI Controls Conversion

### Dropdowns
**Hex pattern:**
```python
use_feature = dropdown("Yes", "No", default="Yes")
```

**marimo pattern:**
```python
@app.cell(hide_code=True)
def _(mo):
    use_feature_input = mo.ui.dropdown(
        options=["Yes", "No"],
        value="Yes",
        label="Use Feature:"
    )
    mo.vstack([
        mo.md("### Settings"),
        use_feature_input
    ])
    return (use_feature_input,)

# Access value in other cells
@app.cell(hide_code=True)
def _(use_feature_input):
    USE_FEATURE = use_feature_input.value == "Yes"
    return (USE_FEATURE,)
```

### Sliders
**marimo pattern:**
```python
@app.cell(hide_code=True)
def _(mo):
    threshold_input = mo.ui.slider(
        start=0,
        stop=100,
        value=50,
        label="Threshold:"
    )
    threshold_input
    return (threshold_input,)
```

## Chart Conversion: Hex to marimo Altair

### About Altair
Hex uses Altair by default for charting, so most Hex charts can be translated directly to marimo with minimal changes. The main difference is using `mo.ui.altair_chart()` instead of displaying the chart object directly.

### Basic Chart Pattern
**Hex (Altair):**
```python
import altair as alt
chart = alt.Chart(df).mark_bar().encode(
    x='category:N',
    y='value:Q',
    color='group:N'
)
chart
```

**marimo (Altair):**
```python
@app.cell(hide_code=True)
def _(alt, df, mo):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensource-observer/insights](https://github.com/opensource-observer/insights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
