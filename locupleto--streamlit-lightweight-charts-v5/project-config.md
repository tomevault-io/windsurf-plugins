---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Frontend Development (TypeScript/React)
```bash
cd lightweight_charts_v5/frontend
npm install                    # Install dependencies
npm run build                  # Build the React component
npm start                      # Start development server on port 3001
```

### Python Package Development
```bash
pip install -e .               # Install package in editable mode
pip install -e .[devel]        # Install with development dependencies
pip install -e .[demo]         # Install with demo dependencies (yfinance, numpy)
```

### Testing
```bash
# Python tests (pytest available in devel extras)
pytest

# Frontend tests
cd lightweight_charts_v5/frontend
npm test
```

### Running Demos
```bash
streamlit run demo/minimal_demo.py    # Basic stock chart example
streamlit run demo/chart_demo.py      # Advanced multi-pane example with indicators
```

### Build and Release Process
```bash
# Clean previous builds
rm -rf build/ dist/ *.egg-info/

# Build frontend first
cd lightweight_charts_v5/frontend
npm install && npm run build
cd ../..

# Build Python package
python -m pip install --upgrade build
python -m build
```

## Architecture Overview

This is a Streamlit component that wraps TradingView's Lightweight Charts v5 library for financial charting.

### Key Components

**Python Layer (`lightweight_charts_v5/`)**:
- `__init__.py`: Main component interface with `lightweight_charts_v5_component()` function
- Handles dev server detection (port 3001) vs production build switching
- Supports both single-pane (`data` param) and multi-pane (`charts` param) configurations

**Frontend Layer (`lightweight_charts_v5/frontend/`)**:
- `LightweightChartsComponent.tsx`: Core React component implementing multi-pane chart logic
- Built with React 16.13.1 and TypeScript
- Uses `lightweight-charts` v5.0.2 library
- Handles complex pane height initialization and window resize race conditions

**Demo Applications (`demo/`)**:
- Multiple example implementations showing various chart types and indicators
- Uses Yahoo Finance data via `yfinance` library

### Development Workflow

The component supports dual-mode development:
1. **Development**: Auto-detects if React dev server is running on port 3001
2. **Production**: Uses built static files from `frontend/build/`

### Key Technical Challenges Solved

1. **Pane Height Management**: Sequential bottom-to-top height setting with async delays
2. **Window Resize Handling**: Debounced resize events (800ms) with disposal state tracking
3. **Chart Disposal**: Proper cleanup to prevent "Object is disposed" errors during paint cycles

### Chart Configuration

Charts support:
- Multiple panes with synchronized time scales
- Various series types: Candlestick, Line, Area, Histogram, Bar
- Overlay indicators (moving averages, etc.)
- Rectangle drawing for support/resistance areas
- Screenshot functionality
- Custom themes and styling
- Google Fonts integration

## Mermaid Diagram Generation

### **When to Generate Diagrams**
When users request visual documentation or illustrations using terms like:
- "draw a diagram"
- "make me a mermaid diagram" 
- "illustrate for me"
- "show me a flowchart"
- "create a diagram showing..."
- Any request for visual representation of architecture, processes, or relationships

### **Standard Diagram Generation Process**
1. **Use the MCP Mermaid service** via `mcp__mermaid-diagram__generate_diagram`
2. **Always specify a descriptive file_name** (required parameter, without extension)
3. **Use dark theme for dark environments** or default theme for light environments
4. **Diagrams are automatically saved** with the specified filename and appropriate extension

### **Updated Implementation Pattern**
```python
# Generate Mermaid diagram with required file_name parameter
diagram_result = mcp__mermaid-diagram__generate_diagram(
    mermaid_code=mermaid_syntax,
    file_name="architecture-overview",  # REQUIRED: descriptive name without extension
    theme="dark",  # Use "dark" for dark environments, "default" for light environments
    format="svg",  # Default: svg, also supports png, pdf
    backgroundColor="transparent"  # Default: transparent (adapts to container theme)
    # Other parameters: width=1920, height=1080, scale=2
)

# File is automatically saved as: architecture-overview.svg
# For SVG format: Content is also returned for immediate embedding
# For PNG/PDF: Only file path is returned
```

### **Diagram Types to Support**
- **Architecture diagrams**: System components, data flow
- **Process flowcharts**: User workflows, business processes  
- **Class diagrams**: Code structure, inheritance relationships
- **Sequence diagrams**: Interaction flows, API calls
- **Entity relationship diagrams**: Database schemas
- **Network diagrams**: System topology, connections

### **Best Practices**
- **Generate appropriate Mermaid syntax** for the requested diagram type
- **Use descriptive filenames** that clearly identify the diagram content
- **Use clear, descriptive node labels** and relationships
- **Keep diagrams focused** - break complex systems into multiple diagrams

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [locupleto/streamlit-lightweight-charts-v5](https://github.com/locupleto/streamlit-lightweight-charts-v5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
