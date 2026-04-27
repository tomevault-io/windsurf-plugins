---
trigger: always_on
description: - **Readability:** Code should be easy to understand for all team members.
---

# SecOps SDK Python Style Guide

# Introduction
# This style guide outlines the coding conventions for Python code in the SecOps SDK.
# It follows the Google Python Style Guide with specific adaptations for this project.
# https://google.github.io/styleguide/pyguide.html

# Key Principles
- **Readability:** Code should be easy to understand for all team members.
- **Maintainability:** Code should be easy to modify and extend.
- **Consistency:** Adhere to a consistent style across all modules.
- **Performance:** While readability is paramount, code should be efficient.
- **Security:** Follow secure coding practices, especially for API handling.

# Code Structure and Formatting
## Line Length
- Maximum line length: 80 characters
- This ensures code is readable in all environments and follows the project's pylintrc configuration

## Indentation
- Use 4 spaces per indentation level (not tabs)
- Indent continued lines by 4 spaces
- Maintain consistent indentation for multi-line statements

## Imports
- Group imports in the following order:
  - Standard library imports
  - Related third party imports
  - Local application/library specific imports
- Sort imports alphabetically within each group
- Use absolute imports for clarity
- Do not use wildcard imports (from x import *)

## Whitespace
- Use vertical whitespace to separate logical sections of code
- No trailing whitespace at the end of lines
- Two blank lines before top-level function and class definitions
- One blank line before method definitions
- Use spaces around operators and after commas

# Naming Conventions
## Variables and Functions
- Use lowercase with underscores (snake_case): `user_name`, `process_data()`
- Function names should be verbs or verb phrases: `fetch_data()`, `validate_input()`

## Constants
- Use uppercase with underscores: `MAX_RETRIES`, `DEFAULT_TIMEOUT`

## Classes
- Use CapWords (CamelCase): `ChronicleClient`, `DataProcessor`
- Class names should be nouns or noun phrases

## Modules and Packages
- Use lowercase with underscores: `data_processing`, `auth_handler`
- Keep module names short and descriptive

# Documentation
## Docstrings
- Use Google style docstrings for all public modules, functions, classes, and methods
- First line should be a concise summary of the object's purpose
- Include detailed descriptions of parameters, return values, attributes, and exceptions
- Example:
  ```python
  def process_data(input_data: dict) -> dict:
      """Process raw data into a structured format.
      
      Args:
          input_data: Dictionary containing raw data to process.
              Must include 'timestamp' and 'value' keys.
              
      Returns:
          Dictionary with processed data containing:
          - 'processed_timestamp': ISO-8601 formatted timestamp
          - 'normalized_value': Value normalized to range [0,1]
          
      Raises:
          ValueError: If required fields are missing from input_data.
      """
  ```

## Comments
- Write clear and concise comments explaining the "why" not just the "what"
- Use comments sparingly and let well-written code document itself where possible
- Begin comments with # and a single space
- Use complete sentences with proper capitalization and punctuation

# Type Hints
- Use type hints for function parameters and return values
- Follow PEP 484 for type hinting syntax
- Use Optional[] for parameters that can be None
- Use Union[] for parameters that can be multiple types
- Use typing module for complex types (Dict, List, Tuple, etc.)

# Error Handling
- Use specific exceptions rather than catching broad exceptions
- Handle exceptions gracefully with informative error messages
- Create custom exception classes for domain-specific errors
- Provide proper context in error messages
- Log errors appropriately

# Testing
- Write unit tests for all public methods and functions
- Follow the test naming pattern: `test_<function_name>_<scenario>`
- Test both normal operation and edge cases
- Mock external dependencies in unit tests
- Use pytest fixtures for common test setup
- Aim for high code coverage

# Security Best Practices
- Never hardcode sensitive information (passwords, API keys, etc.)
- Use proper authentication mechanisms
- Validate all inputs, especially user inputs
- Handle API errors and rate limits gracefully
- Follow secure coding practices for API access

# Example Service Method Code
```python
"""Example service functionality module."""

from typing import Dict, Any, Optional
from datetime import datetime

from secops.chronicle import ChronicleClient
from secops.exceptions import APIError, SecOpsError

# Constants should be UPPER_CASE
DEFAULT_SORT = "ASC"


def example_service(
    client: "ChronicleClient",
    resource_id: str,
    params: Optional[Dict[str, Any]] = None,
) -> Dict[str, Any]:
    """Performs an example service operation with proper error handling.

    Args:
        client: ChronicleClient instance for API requests
        resource_id: Unique identifier for the target resource
        params: Optional parameters to include in the request

    Returns:
        Dictionary containing the processed service response with:
        - status: Operation result status
        - data: The returned resource data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d3kum1d0r1y4100/hydra-sandbox-079b64185dbc](https://github.com/d3kum1d0r1y4100/hydra-sandbox-079b64185dbc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
