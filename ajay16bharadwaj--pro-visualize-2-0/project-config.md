---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session Usage Policy

**If context usage reaches 90%:** immediately (1) update `PRODUCTION_READINESS_PLAN.md` with completed work, (2) save relevant memories, then (3) stop. Do not continue implementing beyond this point. The next session will resume from the plan.

## Overview

Pro-Visualize is a Streamlit-based proteomics data visualization application supporting QC, Quantification, Comparative Analysis, Pathway Enrichment, Dilution Series, and Functional Annotation workflows.

## Running the Application

```bash
# Install dependencies
pip install -r requirements.txt

# Run the Streamlit app
streamlit run app.py
```

The application runs on port 8501 by default and opens automatically in your browser.

## Architecture

### Core Structure

The application follows a modular, tab-based architecture where each analysis type is self-contained:

- **`app.py`**: Main entry point that configures Streamlit and creates top-level tabs
- **`modules/`**: Analysis modules, each with a `render()` function called from app.py
  - Each module defines its own UI/UX and manages session state
  - Module pattern: Class-based with `_upload_data_section()`, `_display_results()`, and `render()` methods
- **`visualizations/`**: Visualizer classes that handle data processing and plot generation
  - Separated from UI logic to maintain clean architecture
  - Each visualizer is initialized with data and column configuration
  - Methods return Plotly/Matplotlib figures or BytesIO buffers
- **`utils/`**: Shared utilities
  - `helpers.py`: Decorators like `@handle_plotting_errors`
  - `caching.py`: Caching functions for expensive operations
  - `data_manager.py`: Data handling utilities
- **`config/`**: Configuration files (currently minimal)

### Module ↔ Visualizer Relationship

Each module instantiates its corresponding visualizer class and stores it in `st.session_state`:

```
modules/quant_module.py (QuantificationTab)
    ↓ creates & stores in session_state
visualizations/quant_visualizer.py (QuantificationVisualizer)
    ↓ returns figures to
modules/quant_module.py
```

**Pattern used throughout:**
1. User uploads data via module UI
2. Module validates input and creates Visualizer instance
3. Visualizer stored in `st.session_state.{module_name}_visualizer`
4. Module displays plots by calling visualizer methods

### Column Configuration Pattern

Modules use configurable column names to support diverse proteomics file formats:

```python
# Users specify column names via text inputs:
protein_col = st.text_input("Protein ID Column", value="Protein")
sample_col = st.text_input("Sample Linking Column", value="Level3")
group_col = st.text_input("Group Column", value="attribute_ExperimentalGroup")

# These are passed to the visualizer constructor:
visualizer = QuantificationVisualizer(
    protein_df, annotation_df,
    protein_col=protein_col,
    sample_col=sample_col,
    group_col=group_col
)
```

For comparative analysis, a `column_config` dict is used instead:

```python
column_config = {
    "protein_id": protein_id_col,
    "sample_id": sample_id_col,
    "fold_change": fold_change_col,
    "fdr": fdr_col,
    ...
}
visualizer = ComparativeVisualizer(protein_df, annotation_df, comparative_df, column_config)
```

### Session State Management

- Each analysis module maintains its own session state namespace
- Pattern: `st.session_state.{module}_visualizer` stores the visualizer instance
- Comparative module also uses `st.session_state.significant_proteins` to share filtering results across tabs
- Enrichment results cached in `st.session_state.enrichment_results`

### Data Flow

1. **File Upload**: Users upload CSV/TSV/TXT files (auto-detected separator with `sep=None, engine='python'`)
2. **Data Validation**: Visualizer `__init__` validates required columns exist
3. **Processing**: Visualizer methods process data (filtering, transformations, clustering, etc.)
4. **Visualization**: Methods return Plotly figures, Matplotlib figures, or BytesIO buffers
5. **Display**: Module renders using `st.plotly_chart()`, `st.pyplot()`, or `st.image()`

### QC Module Structure

The QC module has a unique sub-tab architecture:

```
modules/qc_module.py
    ├── modules/qc_tabs/dia_qc_tab.py (DiaQcTab)
    │   └── visualizations/DiaQcVisualizer.py
    └── modules/qc_tabs/targeted_qc_tab.py (TargetedQcTab)
        └── visualizations/targettedQCVisualization.py
```

## Key Implementation Details

### Plot Generation Patterns

**Interactive Plotly plots** (quantification, comparative):
```python
def plot_something(self):
    fig = px.scatter(data, x='col1', y='col2', color='group')
    fig.update_layout(height=600, title="Plot Title")
    return fig
```

**Static Matplotlib plots** (heatmaps, dendrograms):
```python
def plot_heatmap(self):
    fig, ax = plt.subplots(figsize=(10, 8))
    # ... plotting logic ...
    buf = BytesIO()
    plt.savefig(buf, format="png", bbox_inches="tight")
    buf.seek(0)
    plt.close(fig)
    return buf
```

### Error Handling

Use the `@handle_plotting_errors` decorator for visualizer methods to gracefully handle exceptions in the UI:

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajay16bharadwaj/Pro-Visualize-2.0](https://github.com/ajay16bharadwaj/Pro-Visualize-2.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
