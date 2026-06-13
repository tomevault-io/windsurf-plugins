---
trigger: always_on
description: > Institutional memory for Claude Code sessions. Update this file whenever Claude makes mistakes or learns something new about the project.
---

# CLAUDE.md - Docx Parser Converter

> Institutional memory for Claude Code sessions. Update this file whenever Claude makes mistakes or learns something new about the project.

---

## Project Overview

A dual-implementation DOCX parser and converter:
- **Python**: Production-ready library (`docx_parser_converter_python/`)
- **TypeScript**: Browser-compatible version (`docx_parser_converter_ts/`)

Converts `.docx` files to HTML or plain text while preserving formatting, styles, lists, and tables.

---

## Tech Stack

### Python Implementation
- **Language**: Python 3.10+
- **Package Manager**: PDM
- **Core Dependencies**: lxml, pydantic v2
- **Testing**: pytest (1,493 tests)
- **Type Checking**: pyright (basic mode)
- **Linting**: ruff

### TypeScript Implementation
- **Language**: TypeScript 5.2+
- **Package Manager**: npm
- **Core Dependencies**: @xmldom/xmldom, jszip
- **Build**: Vite (ES, UMD, IIFE outputs)
- **Linting**: ESLint with typescript-eslint

---

## Project Structure

```
/
├── docx_parser_converter_python/    # Main Python implementation
│   ├── core/                        # Infrastructure (DocxReader, etc.)
│   ├── models/                      # Pydantic data models
│   ├── parsers/                     # XML-to-model parsers
│   ├── converters/                  # HTML/Text converters
│   ├── tests/unit/                  # Unit tests
│   └── api.py                       # Public API
├── docx_parser_converter_ts/        # TypeScript implementation
│   └── src/
├── fixtures/                        # Test data
│   ├── test_docx_files/             # Input .docx files + expected outputs
│   └── tagged_tests/                # Tagged test DOCX files (Test #N format)
└── docs/                            # Documentation
```

---

## Essential Commands

### Python (run from `docx_parser_converter_python/`)

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/unit/converters/html/test_html_converter.py -v

# Run tests matching pattern
pytest -k "test_list" -v

# Type checking
pyright

# Linting
ruff check .

# Auto-fix lint issues
ruff check . --fix

# Format code
ruff format .
```

### TypeScript (run from `docx_parser_converter_ts/`)

```bash
# Development server
npm run dev

# Build
npm run build

# Lint
npm run lint
```

---

## Architecture Patterns

### Data Flow
```
DOCX File → DocxReader → XML Parts → Parsers → Pydantic Models → Converters → Output
```

### Key Principles

1. **Type-First Approach**: All data flows through Pydantic v2 models
2. **Immutable Models**: Use `frozen=True` on Pydantic models
3. **Raw Values**: Models store raw XML values (no unit conversion during parsing)
4. **Style Resolution**: Hierarchical inheritance (direct → style → defaults)
5. **Parser Separation**: One parser per XML part (document, styles, numbering)

### Public API

```python
from docx_parser_converter_python import docx_to_html, docx_to_text

# Convert to HTML
html = docx_to_html("document.docx")

# Convert to text
text = docx_to_text("document.docx")

# With configuration
from docx_parser_converter_python import ConversionConfig, StyleMode
config = ConversionConfig(style_mode=StyleMode.INLINE)
html = docx_to_html("document.docx", config=config)
```

---

## Code Style Guidelines

### Python

1. **Type hints everywhere** - All function parameters and returns must be typed
2. **Pydantic for data** - Use Pydantic models, not dataclasses or dicts
3. **Private methods** - Prefix with `_` for internal methods
4. **No magic numbers** - Use constants or enums
5. **Docstrings** - Required for public API functions only

### Naming Conventions

```python
# Classes: PascalCase
class StyleResolver:
    pass

# Functions/methods: snake_case
def parse_document_xml():
    pass

# Constants: SCREAMING_SNAKE_CASE
DEFAULT_FONT_SIZE = 11

# Private: leading underscore
def _internal_helper():
    pass
```

### Import Order

```python
# 1. Standard library
from typing import Optional
from pathlib import Path

# 2. Third-party
from lxml import etree
from pydantic import BaseModel

# 3. Local imports
from .models import Paragraph
from .core import DocxReader
```

---

## Common Mistakes to Avoid

### 1. Forgetting to Handle None/Optional Values

```python
# BAD
def get_font_size(run) -> int:
    return run.properties.font_size  # May be None!

# GOOD
def get_font_size(run) -> Optional[int]:
    if run.properties and run.properties.font_size:
        return run.properties.font_size
    return None
```

### 2. Modifying Frozen Pydantic Models

```python
# BAD - Will raise error
model.field = new_value

# GOOD - Create new instance
new_model = model.model_copy(update={"field": new_value})
```

### 3. Not Using the Style Resolver

```python
# BAD - Direct property access misses inheritance
font_size = paragraph.properties.font_size

# GOOD - Use resolver for proper inheritance
resolved = style_resolver.resolve_paragraph_properties(paragraph)
font_size = resolved.font_size
```

### 4. Hardcoding XML Namespaces

```python
# BAD
element.find("{http://schemas.openxmlformats.org/wordprocessingml/2006/main}p")

# GOOD - Use constants from core/constants.py
from core.constants import NAMESPACES

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omer-go/docx-parser-converter](https://github.com/omer-go/docx-parser-converter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
