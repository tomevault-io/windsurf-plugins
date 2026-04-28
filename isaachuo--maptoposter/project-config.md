---
trigger: always_on
description: This file provides essential information for agentic coding agents working in the maptoposter repository.
---

# AGENTS.md - City Map Poster Generator

This file provides essential information for agentic coding agents working in the maptoposter repository.

## Project Overview

A Python-based web application that generates beautiful, minimalist map posters for any city in the world. Built with Gradio for the web interface, OSMnx for mapping data, and Matplotlib for rendering.

**Tech Stack:**
- Python 3.12+
- Gradio (web UI)
- OSMnx (street network analysis)
- Matplotlib (map rendering)
- GeoPandas (geospatial data)
- Nominatim (geocoding)

## Build/Test/Lint Commands

### Environment Setup
```bash
# Install dependencies with uv (recommended)
uv sync

# Alternative: with pip
pip install -r requirements.txt  # (if available)
```

### Running the Application
```bash
# Start the web interface
uv run python app.py

# Or use the restart script (handles port conflicts)
bash restart.sh

# The application runs on http://localhost:7860
```

### Running Tests
```bash
# Run the main test file
python test_logic.py

# Run individual test functions
python -c "from test_logic import test_china_hierarchy; test_china_hierarchy()"
python -c "from test_logic import test_manual_coordinates; test_manual_coordinates()"
```

### Development Utilities
```bash
# Verify city data integrity
python verify_data.py

# Create map poster directly (CLI)
python create_map_poster.py --city "Beijing" --theme japanese_ink --output-format png
```

## Code Style Guidelines

### General Principles
- **Bilingual Support**: All user-facing text should support both English (en) and Chinese (cn)
- **File Encoding**: Use UTF-8 encoding with BOM for Chinese characters: `# -*- coding: utf-8 -*-`
- **Function Documentation**: Use docstrings for all public functions
- **Error Handling**: Provide meaningful error messages in both languages when possible

### Import Organization
```python
# Standard library imports first
import os
import json
import sys
from datetime import datetime

# Third-party imports next
import gradio as gr
import osmnx as ox
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from geopy.geocoders import Nominatim

# Local imports last
from cities_data import get_countries, get_provinces, get_cities
from create_map_poster import generate_map_poster
```

### Naming Conventions
- **Files**: `snake_case.py` (e.g., `cities_data.py`, `test_logic.py`)
- **Variables/Functions**: `snake_case` (e.g., `get_available_themes()`, `motorway_color`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `THEMES_DIR`, `LAYER_KEYS`)
- **Classes**: `PascalCase` (rare in this codebase, but follow convention if used)

### Type Hints
```python
def generate_output_filename(city: str, theme_name: str, output_format: str) -> str:
    """Generate unique output filename with city, theme, and datetime."""
    pass

def get_theme_choices(lang: str = "en") -> list[tuple[str, str]]:
    """Return a list of (Display Name, Internal Name) tuples for themes."""
    pass
```

### String Patterns
- **Language Keys**: Use `lang="en"` or `lang="cn"` parameters
- **Layer Constants**: Define parallel arrays for different languages
```python
LAYERS_EN = ["Motorway", "Primary Roads", "Secondary Roads", "Water", "Parks"]
LAYERS_CN = ["高速公路", "主干道", "次干道", "水域", "公园"]
LAYER_KEYS = ["motorway", "primary", "secondary", "water", "parks"]
```

### Data Structures
- **City Hierarchy**: Use tuples for display/internal name pairs: `[("Display Name", "internal_name"), ...]`
- **Theme Files**: JSON format with descriptive names: `{"name": "Theme Name", "bg": "#FFFFFF", ...}`
- **Coordinates**: Use `(lat, lon)` tuple format

## Directory Structure

```
maptoposter/
├── app.py                 # Main Gradio web interface
├── create_map_poster.py   # Core poster generation logic
├── cities_data.py         # City/province/district data management
├── test_logic.py          # Test suite for data logic
├── verify_data.py         # Data integrity verification
├── src/maptoposter/       # Package structure (minimal)
├── themes/                # JSON theme configuration files
├── fonts/                 # Font files (Chinese and English)
├── posters/               # Generated poster outputs
├── public/                # Static assets and demo images
└── restart.sh            # Development restart script
```

## Common Patterns

### Language Support
```python
def translate(text: str, lang: str = "en") -> str:
    """Simple translation wrapper."""
    translations = {
        "en": {"City": "City", "Theme": "Theme"},
        "cn": {"City": "城市", "Theme": "主题"}
    }
    return translations.get(lang, {}).get(text, text)
```

### File Path Handling
```python
THEMES_DIR = "themes"
FONTS_DIR = "fonts"
POSTERS_DIR = "posters"

# Always check existence
if not os.path.exists(THEMES_DIR):
    return []
```

### Gradio Interface Patterns
```python
def create_interface():
    with gr.Blocks() as interface:
        gr.Markdown("# City Map Poster Generator")
        
        with gr.Row():
            city_dropdown = gr.Dropdown(label="City", choices=[])
            theme_dropdown = gr.Dropdown(label="Theme", choices=[])
        
        generate_btn = gr.Button("Generate Poster")
        output_gallery = gr.Gallery()
        
        # Event handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IsaacHuo/maptoposter](https://github.com/IsaacHuo/maptoposter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
