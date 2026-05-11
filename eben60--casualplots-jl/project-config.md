---
trigger: always_on
description: **CasualPlots.jl** is a GUI-based plotting application for Julia which is positioned in the middle ground between purely script-based plotting and standalone GUI plotting applications. Target users are experimental scientists and engineers needing quick visualization without memorizing syntax. Aims to cover 60-80% of common 2D plotting needs (Scatter/Line/BarPlot, basic formatting).
---

# CasualPlots.jl - AI Agent Technical Reference

## Package Overview
**CasualPlots.jl** is a GUI-based plotting application for Julia which is positioned in the middle ground between purely script-based plotting and standalone GUI plotting applications. Target users are experimental scientists and engineers needing quick visualization without memorizing syntax. Aims to cover 60-80% of common 2D plotting needs (Scatter/Line/BarPlot, basic formatting).

## Core Architecture

### JavaScript Conventions
*   **External Logic**: All non-trivial JavaScript logic must be placed in `src/javascripts.js` and namespaced under `window.CasualPlots`.
*   **Inline Minimization**: Inline JS in Julia files (`js"..."`) should be restricted to simple calls to these external functions or mandatory one-liners.
*   **Loading**: The `javascripts.js` file is read and injected as a script tag in the main application layout (`app.jl`).

### Technology Stack
*   **[Bonito.jl](https://github.com/SimonDanisch/Bonito.jl)**: Web-based reactive GUI framework
*   **[WGLMakie](https://github.com/MakieOrg/Makie.jl)**: WebGL-based plotting backend
*   **[AlgebraOfGraphics.jl](https://github.com/MakieOrg/AlgebraOfGraphics.jl)**: Declarative plot specification (all plots built using AoG)
*   **[DataFrames.jl](https://github.com/JuliaData/DataFrames.jl)**: Data handling
*   **[Observables.jl](https://github.com/JuliaGizmos/Observables.jl)**: Reactive state management
*   **[Electron.jl](https://github.com/davidanthoff/Electron.jl)**: Window hosting 
*   **[CSV.jl](https://github.com/JuliaData/CSV.jl)** / **[XLSX.jl](https://github.com/felipenoris/XLSX.jl)**: File I/O via Package Extensions

### File Structure (src/)

```
CasualPlots.jl                  # Main module, exports casualplots_app()
app.jl                          # Main app entry point (casualplots_app function)
app_state.jl                    # Application state initialization (Observables)
css_styles.css                  # Global CSS styles for all UI components
javascripts.js                  # Global JavaScript functions (namespaced window.CasualPlots)

# Core Logic
plotting.jl                     # Plot generation using AlgebraOfGraphics
setup_callbacks.jl              # Core reactive callbacks (do_replot, source, format, DataFrame)
label_update_callbacks.jl       # Label text field callbacks
dropdowns_setup.jl              # Dropdown menu creation (X, Y, DataFrame)

# UI Components (ui_*.jl)
ui_tabs.jl                      # Tab component + create_tab_content wiring
ui_layout.jl                    # assemble_layout - main pane grid construction
ui_table.jl                     # Table display with info header
ui_help_section.jl              # Mouse controls help text
ui_source_tab.jl                # Source selection UI (Array/DataFrame modes)
ui_format_tab.jl                # Format controls UI (plot type, legend, labels)
ui_open_tab.jl                  # File open tab UI
ui_save_tab.jl                  # Save tab UI
ui_modal_dialog.jl              # Modal dialog component

# Control Panel
create_control_panel_ui.jl      # Control panel UI construction

# Data Handling
collect_data.jl                 # Data collection from Main module
preprocess_dataframes.jl        # Data frame normalization and validation
read_from_file.jl               # File reading logic (CSV/XLSX) and loading callbacks
file_reading_options.jl         # Options processing for file reading
create_demo_data.jl             # Demo data generation

# Save/Export
save_plot.jl                    # Plot saving functionality (CairoMakie backend)

# Other
electron.jl                     # Electron window integration (show kwarg for hidden windows)
FileDialogWorkAround.jl         # Cross-platform file dialog utilities
extensions.jl                   # Package extensions loader
precompile.jl                   # PrecompileTools workload for reducing TTFP

scripts/                        # Example/demo scripts
../ext/                         # Package Extensions (ReadCSV_Ext.jl, ReadXLSX_Ext.jl)
```

### Reactive State Architecture

The application uses a reactive `state` NamedTuple with `Observables.jl` for all UI state management.
See [Reactive State Architecture](AGENTS_more_info/specific_issues/reactive_state_architecture.md) for the full state structure and output observables documentation.

### Developer Diagrams

Diagrams are in the linked files:

- [High-Level User Flow](AGENTS_more_info/Mermaid/high-level_user_flow.md)
- [Callback Execution Sequence](AGENTS_more_info/Mermaid/callback_execution_sequence.md)
- [State Transition Map](AGENTS_more_info/Mermaid/state_transition_map.md)


### Critical Implementation Patterns

#### 1. Source Selection & Plotting Flow
Both **X,Y Source**, **DataFrame Source**, and **Open File** modes feed into the plotting pipeline.

**A. X,Y Source Selection:**
1.  **Step 1: X Selection** (`setup_x_callback`)
    - User selects X variable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eben60/CasualPlots.jl](https://github.com/Eben60/CasualPlots.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
