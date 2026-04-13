---
trigger: always_on
description: **Note**: This document applies only to the legacy Python implementation preserved in the `legacy/` directory and the Python-based XSD model generation tools. The primary project implementation is in Rust.
---


# Python Development Standards for Legacy Code and XSD Tooling

**Note**: This document applies only to the legacy Python implementation preserved in the `legacy/` directory and the Python-based XSD model generation tools. The primary project implementation is in Rust.

## Scope of Python Usage

Python is used in this project for:

1. **Legacy Implementation Reference**: Original Python code in `legacy/` directory (read-only)
2. **XSD Model Generation**: Using xsdata-pydantic to generate type-safe models from XSD schema
3. **Schema Validation Tooling**: Python scripts for validating generated XML against XSD
4. **Development Utilities**: Build and validation scripts that complement the Rust toolchain

## Project Setup and Dependencies

### Package Management

- **Primary**: Use **UV** package manager for modern dependency management
- **Python Version**: Python 3.10+ required, 3.13+ recommended for best performance
- **Virtual Environment**: Always use isolated environments for Python dependencies

```bash
# Install UV if not available
curl -LsSf https://astral.sh/uv/install.sh | sh

# Setup project with Python dependencies
uv sync --extra dev

# Install xsdata-pydantic for XSD model generation
uv add xsdata-pydantic

# Install additional development tools
uv add --dev mypy black isort pytest
```

### Core Dependencies

**XSD and Validation Tools:**

- **xsdata-pydantic**: For XSD-based model generation with Pydantic validation
- **lxml**: For XML processing and validation
- **pydantic**: For data validation and serialization (used by generated models)
- **xmlschema**: Alternative XSD validation library

**Development Tools:**

- **mypy**: Static type checking
- **black**: Code formatting (119 character line length to match project)
- **isort**: Import organization
- **pytest**: Testing framework

## Code Style and Quality

### Formatting Standards

```toml
# pyproject.toml - Python configuration
[tool.black]
line-length = 119  # Match Rust project standards
target-version = ['py310']
include = '\.pyi?$'
extend-exclude = '''
/(
  # Exclude generated model files
  opnsense/models/.*\.py
)/
'''

[tool.isort]
profile = "black"
line_length = 119
skip_glob = ["opnsense/models/*"]

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
exclude = [
    "^opnsense/models/.*\\.py$",  # Skip generated models
    "^legacy/.*\\.py$",           # Skip legacy code
]
```

### Type Safety Requirements

All Python code must use comprehensive type hints:

```python
from typing import Optional, List, Dict, Any, Union
from pathlib import Path
import xml.etree.ElementTree as ET

def validate_opnsense_config(
    xml_file: Path,
    xsd_schema: Path,
    strict: bool = True
) -> tuple[bool, List[str]]:
    """Validate OPNsense XML configuration against XSD schema.

    Args:
        xml_file: Path to the XML configuration file
        xsd_schema: Path to the XSD schema file
        strict: Whether to enforce strict validation rules

    Returns:
        Tuple of (is_valid, list_of_errors)

    Raises:
        FileNotFoundError: If xml_file or xsd_schema doesn't exist
        XMLSyntaxError: If XML is malformed
    """
    errors: List[str] = []

    try:
        # Implementation with proper error handling
        from lxml import etree

        with open(xsd_schema, 'r') as schema_file:
            schema_doc = etree.parse(schema_file)
            schema = etree.XMLSchema(schema_doc)

        with open(xml_file, 'r') as xml_doc:
            xml_tree = etree.parse(xml_doc)

        is_valid = schema.validate(xml_tree)
        if not is_valid:
            errors = [str(error) for error in schema.error_log]

        return is_valid, errors

    except FileNotFoundError as e:
        raise FileNotFoundError(f"Required file not found: {e}")
    except etree.XMLSyntaxError as e:
        raise etree.XMLSyntaxError(f"Malformed XML: {e}")
```

## XSD Model Generation Integration

### xsdata-pydantic Usage

```python
# scripts/generate_models.py - Model generation script
from pathlib import Path
import subprocess
import sys
from typing import List, Optional

def generate_pydantic_models(
    xsd_file: Path,
    output_package: str = "opnsense.models",
    config_file: Optional[Path] = None
) -> bool:
    """Generate Pydantic models from XSD schema using xsdata.

    Args:
        xsd_file: Path to the XSD schema file
        output_package: Python package name for generated models
        config_file: Optional xsdata configuration file

    Returns:
        True if generation successful, False otherwise
    """
    if not xsd_file.exists():
        print(f"❌ XSD file not found: {xsd_file}")
        return False

    cmd: List[str] = [
        "xsdata",
        str(xsd_file),
        "--output", "pydantic",
        "--package", output_package,
        "--structure-style", "single-package"
    ]

    if config_file and config_file.exists():
        cmd.extend(["--config", str(config_file)])

    try:
        result = subprocess.run(cmd, check=True, capture_output=True, text=True)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EvilBit-Labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
