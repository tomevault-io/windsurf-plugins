---
trigger: always_on
description: This file provides guidance for agentic coding assistants working on this repository.
---

# AGENTS.md

This file provides guidance for agentic coding assistants working on this repository.

## Build/Lint/Test Commands

**Note:** This project currently has no automated testing or linting infrastructure configured.

### Running Tests
```bash
# No test framework configured. To add tests:
# 1. Install pytest: pip install pytest pytest-cov
# 2. Create test files: scripts/test_*.py
# 3. Run all tests: pytest scripts/
# 4. Run single test: pytest scripts/test_module.py::test_function

# To test the main CLI manually:
python scripts/main.py generate --help
python scripts/main.py list-scenarios
python scripts/main.py list-styles --scenario portrait
```

### Linting
```bash
# No linter configured. Recommended tools:
# Install pylint: pip install pylint
# Run lint: pylint scripts/
# Install black: pip install black
# Format code: black scripts/
# Install mypy: pip install mypy
# Type check: mypy scripts/
```

### Installation
```bash
# Install dependencies
pip install -r requirements.txt

# Set API key (required for operation)
export ARK_API_KEY="your_api_key_here"

# Mock mode for testing without API
export MOCK_API="true"
```

## Code Style Guidelines

### File Structure
- Entry point: `scripts/main.py` (CLI with argparse subcommands)
- Core modules: `scripts/config.py`, `scripts/interaction.py`, `scripts/image_generator.py`, `scripts/scenario_handlers.py`
- Data files: `data/*.json` (scenarios, styles, poses, templates, characters)
- Output: `output/images/`, `temp/`, `logs/` (auto-generated)

### Import Order
1. Standard library (sys, os, json, time, argparse, pathlib)
2. Third-party libraries (requests, cv2, numpy, PIL)
3. Local imports (from config import config, from interaction import InteractionManager)

```python
import sys
import os
import json
from pathlib import Path

import requests
import cv2
import numpy as np

from config import config
from interaction import InteractionManager
```

### Formatting
- Indentation: 4 spaces
- Strings: Single quotes for code, double quotes for user-facing text
- Line length: Keep readable, typically under 100-120 characters
- Blank lines: Between methods and logical sections
- Use shebang: `#!/usr/bin/env python3` at top of main.py

### Type Hints
Use type hints from `typing` module in function signatures:
```python
from typing import List, Dict, Optional

def generate_images(self, photos: List[str], count: int) -> Optional[List[str]]:
    """Generate images with type hints for parameters and return values"""
    pass
```

### Naming Conventions
- **Classes**: PascalCase (`Config`, `InteractionManager`, `ImageGenerator`)
- **Functions/Methods**: snake_case (`generate_single_image`, `load_config`)
- **Variables**: snake_case (`user_photo_path`, `image_count`)
- **Constants**: UPPER_SNAKE_CASE (`DEFAULT_DATA_DIR`, `DEFAULT_TEMP_DIR`)
- **Private methods**: Leading underscore (`_load_state`, `_save_config`)
- **Boolean variables**: Prefix with `is_`/`has_` (`is_valid`, `has_error`)

### Error Handling
- Use specific exception types in try-except blocks
- Return None on errors for functions that return values
- Print error messages to stdout with emoji prefixes (❌, ⚠️, ✅)
- Catch specific exceptions: `json.JSONDecodeError`, `ValueError`, `IOError`, `requests.exceptions.RequestException`
- Always validate file paths with `Path(path).exists()`

```python
try:
    with open(file_path, 'r', encoding='utf-8') as f:
        data = json.load(f)
except json.JSONDecodeError as e:
    print(f"❌ Invalid JSON: {e}")
    return None
except IOError as e:
    print(f"❌ File error: {e}")
    return None
```

### Documentation
- Use triple double-quotes for docstrings
- Document classes and public methods
- Include Args and Returns sections for complex functions
- Add inline comments for complex logic
- Use emojis in user-facing print statements (📷, ✅, ❌, ⚠️)

```python
def generate_single_image(self, user_photo: str, character: Dict, index: int) -> Optional[str]:
    """Generate a single image with the given character using Seedream 4.5 API
    
    Args:
        user_photo: Path to user's reference photo
        character: Character dictionary with name, prompt, scene
        index: Image index for filename generation
    
    Returns:
        Path to generated image file, or None if generation fails
    """
    pass
```

### Path Handling
- Always use `pathlib.Path` for file operations
- Use relative paths in config, resolve to absolute paths at runtime
- Ensure directories exist with `mkdir(parents=True, exist_ok=True)`
- Use `str(path)` when passing to external functions expecting strings

```python
from pathlib import Path

def get_output_path(self, filename: str) -> Path:
    output_dir = Path(self.config.config["paths"]["output_dir"]) / "images"
    output_dir.mkdir(parents=True, exist_ok=True)
    return output_dir / filename
```

### API Integration
- Handle timeouts (default 120s for image generation)
- Implement retry logic for transient failures
- Use mock mode for testing without API calls
- Validate API responses before processing
- Rate limit between requests (2-3 second delay)

```python
# Mock mode for testing
self.mock_mode = os.getenv("MOCK_API", "false").lower() == "true"

# API request with timeout and error handling
try:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [songjiayang/photo-studio-skill](https://github.com/songjiayang/photo-studio-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
