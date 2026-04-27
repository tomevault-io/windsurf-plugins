---
trigger: always_on
description: This document contains coding standards and best practices that must be followed for all code development. These rules prioritize maintainability, simplicity, and modern Python development practices.
---

# Claude Coding Rules

## Overview
This document contains coding standards and best practices that must be followed for all code development. These rules prioritize maintainability, simplicity, and modern Python development practices.

## Core Principles
- Write code with minimal complexity for maximum maintainability and clarity
- Choose simple, readable solutions over clever or complex implementations
- Prioritize code that any team member can confidently understand, modify, and debug

## Technology Stack

### Package Management
- Always use `uv` and `pyproject.toml` for package management
- Never use `pip` directly

### Modern Python Libraries
- **Data Processing**: Use `polars` instead of `pandas`
- **Web APIs**: Use `fastapi` instead of `flask`
- **Code Formatting/Linting**: Use `ruff` for both linting and formatting
- **Type Checking**: Use `mypy` - type checks have become actually useful and should be part of CI/CD
- **Performance**: Leverage modern CPython improvements - CPython is now much faster

## Code Style Guidelines

### Function Structure
- All internal/private functions must start with an underscore (`_`)
- Private functions should be placed at the top of the file, followed by public functions
- Functions should be modular, containing no more than 30-50 lines
- Use two blank lines between function definitions
- One function parameter per line for better readability

### Type Annotations
- Use clear type annotations for all function parameters
- One function parameter per line for better readability
- Example:
  ```python
  def process_data(
      input_file: str,
      output_format: str,
      validate: bool = True
  ) -> dict:
      pass
  ```

### Type Hints for Optional Parameters
- Always use `Optional[type]` for parameters that can be None
- Be explicit about optional parameters, especially when they have special meanings:
  ```python
  from typing import Optional, List
  
  def process_samples(
      sample_size: Optional[int] = None,  # None means use default
      language: Optional[str] = None      # None means no filtering
  ) -> List[dict]:
      """Process dataset samples.
      
      Args:
          sample_size: Number of samples. None uses default, 0 means all.
          language: Language filter. None means all languages.
      """
      if sample_size == 0:
          # Special case: process all samples
          return process_all()
      elif sample_size is None:
          # Use default sample size
          sample_size = DEFAULT_SAMPLE_SIZE
          
      # Process with explicit sample size
      return process_with_size(sample_size)
  ```

### Class Definitions with Pydantic
- Consider using Pydantic BaseModel for all class definitions to leverage validation, serialization, and other powerful features
- Pydantic provides automatic validation, type coercion, and serialization capabilities
- Example:
  ```python
  from pydantic import BaseModel, Field, validator
  from typing import Optional
  
  class UserConfig(BaseModel):
      """User configuration settings."""
      
      username: str = Field(..., min_length=3, max_length=50)
      email: str = Field(..., regex=r'^[\w\.-]+@[\w\.-]+\.\w+$')
      timeout_seconds: int = Field(default=30, ge=1, le=300)
      debug_enabled: bool = False
      
      @validator('username')
      def username_alphanumeric(cls, v: str) -> str:
          if not v.replace('_', '').isalnum():
              raise ValueError('Username must be alphanumeric')
          return v.lower()
  ```

### Main Function Pattern
- The main function should act as a control flow orchestrator
- Parse command line arguments and delegate to other functions
- Avoid implementing business logic directly in main()

### Command-Line Interface Design
When creating CLI applications:

1. **Use argparse with comprehensive help**:
   ```python
   parser = argparse.ArgumentParser(
       description="Clear description of what the tool does",
       formatter_class=argparse.RawDescriptionHelpFormatter,
       epilog="""
   Example usage:
       # Basic usage
       uv run python -m module --param value
       
       # With environment variable
       export PARAM=value
       uv run python -m module
   """
   )
   ```

2. **Support both CLI args and environment variables**:
   ```python
   def _get_config_value(cli_value: Optional[str] = None) -> str:
       if cli_value:
           return cli_value
       
       env_value = os.getenv("CONFIG_VAR")
       if env_value:
           return env_value
       
       raise ValueError("Value must be provided via --param or CONFIG_VAR env var")
   ```

3. **Provide sensible defaults**:
   ```python
   parser.add_argument(
       "--sample-size",
       type=int,
       help=f"Number of samples (default: {DEFAULT_SIZE}). Use 0 for all",
   )
   ```

4. **Use special values for "all" options**:
   ```python
   if sample_size == 0 or sample_size is None:
       # Process entire dataset
   else:
       # Process sample
   ```

### Imports
- Write imports as multi-line imports for better readability
- Example:
  ```python
  from .services.output_formatter import (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aarora79) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
