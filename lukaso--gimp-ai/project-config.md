---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with code in this repository.
---

# Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

## Project Overview

**GIMP AI Plugin** is a Python plugin for GIMP 3.0+ that integrates AI image generation capabilities directly into GIMP. This is a beta release (v0.8) supporting OpenAI's image generation models for inpainting, image generation, and layer compositing.

### Key Features
- **AI Inpainting**: Fill selected areas with AI-generated content using text prompts and selection masks
- **AI Image Generation**: Create new images from text descriptions as new layers
- **AI Layer Composite**: Intelligently blend AI content into existing images
- **Zero External Dependencies**: Uses only Python standard library + GIMP APIs

## Technology Stack

### Languages & Frameworks
- **Python 3.x**: Primary development language (compatible with GIMP's Python interpreter)
- **GIMP 3.0+ API**: Using GObject Introspection (gi) bindings
- **GTK 3.0**: For user interface dialogs and widgets
- **GEGL**: For image processing operations

### Key Libraries (all via gi bindings)
- `gi.repository.Gimp` - GIMP plugin API
- `gi.repository.GimpUi` - GIMP UI components
- `gi.repository.Gtk` - GTK UI toolkit
- `gi.repository.Gegl` - Image processing operations
- Standard library only: `urllib`, `json`, `base64`, `tempfile`, `ssl`

### External APIs
- **OpenAI Image API** (`https://api.openai.com/v1/images/...`)
  - Generations endpoint for creating images
  - Edits endpoint for inpainting

## Architecture

### Core Files

1. **`gimp-ai-plugin.py`** (Main plugin, ~4000 lines)
   - `GimpAIPlugin` class: Main plugin entry point
   - Procedure registration (3 procedures: inpainting, generation, composite)
   - Configuration management (API key storage)
   - UI dialogs (GTK-based settings and progress)
   - OpenAI API communication
   - Image processing pipeline

2. **`coordinate_utils.py`** (Pure math utilities, ~600 lines)
   - **Pure functions with no GIMP dependencies** (unit testable)
   - Coordinate transformations for context extraction
   - Mask calculations
   - Optimal shape selection for OpenAI API (1024×1024, 1536×1024, 1024×1536)
   - Padding and scaling algorithms
   - See `ALGORITHMS.md` for detailed algorithm documentation

3. **`install.py`** / **`install_simple.py`**
   - Installation scripts for copying files to GIMP plugin directories

### Key Design Principles

- **Separation of Concerns**: Pure coordinate math is isolated in `coordinate_utils.py`
- **Stateless Operations**: New layers are created for all results (non-destructive)
- **No External Dependencies**: Only standard library + GIMP APIs to simplify installation
- **Error Resilience**: Graceful handling of API failures and user cancellation

## Coding Guidelines

### Python Style
- **PEP 8 compliant**: Follow standard Python conventions
- **Docstrings**: All functions should have clear docstrings with Args/Returns
- **Type hints**: Not currently used, but acceptable to add for new code
- **Indentation**: 4 spaces (no tabs)
- **Line length**: Prefer ~100 chars, but no strict limit

### GIMP API Patterns
```python
# Always use gi bindings with version requirements
import gi
gi.require_version("Gimp", "3.0")
gi.require_version("GimpUi", "3.0")
from gi.repository import Gimp, GimpUi

# Plugin class must inherit from Gimp.PlugIn
class MyPlugin(Gimp.PlugIn):
    def do_query_procedures(self):
        return ["my-procedure-name"]
    
    def do_create_procedure(self, name):
        procedure = Gimp.ImageProcedure.new(...)
        return procedure
```

### Configuration Management
- API keys stored in `~/.config/gimp-ai-plugin/config.json`
- Fallback locations: `~/.gimp-ai-config.json`, local `config.json`
- Never commit real API keys to the repository

### Error Handling
- Always wrap API calls in try/except blocks
- Provide user-friendly error messages via `Gimp.message()`
- Handle cancellation via `self._cancel_requested` flag
- Log detailed errors for debugging (can include in error dialogs)

## Testing

### Current Test Infrastructure

1. **`test_plugin.py`**: Basic validation tests
   - Configuration file validation
   - Dependency checks (optional)
   - Module import tests

2. **`test_minimal.py`**: Minimal plugin loading test

3. **`tests/`**: Directory for additional unit tests

### Testing Guidelines

- **Unit tests**: Pure functions in `coordinate_utils.py` should have unit tests
- **Integration tests**: Not currently implemented (would require GIMP environment)
- **Manual testing**: Required for UI and GIMP integration
  - Test all three procedures: inpainting, generation, composite
  - Test with various image sizes and aspect ratios
  - Test cancellation behavior
  - Test error handling (invalid API keys, network failures)

### Running Tests
```bash
# Unit tests (for coordinate_utils.py)
python3 -m pytest tests/

# Manual testing in GIMP
# 1. Copy plugin files to GIMP plugin directory
# 2. Restart GIMP
# 3. Check Filters → AI menu
# 4. Test each feature with various inputs
```

## Build & Validation

### Installation Process
```bash
# The plugin has no build step - just copy files:
mkdir -p ~/.config/GIMP/3.0/plug-ins/gimp-ai-plugin/
cp gimp-ai-plugin.py ~/.config/GIMP/3.0/plug-ins/gimp-ai-plugin/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukaso/gimp-ai](https://github.com/lukaso/gimp-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
