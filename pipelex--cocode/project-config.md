---
trigger: always_on
description: Provides methods for resizing, converting, and optimizing images.
---

Concatenation
# Coding Standards & Best Practices

This document outlines the core coding standards, best practices, and quality control procedures for the codebase.

## Type Hints

1. **Always Use Type Hints**
   - Every function parameter must be typed
   - Every function return must be typed
   - Use type hints for all variables where type is not obvious
   - Use types with Uppercase first letter (Dict[], List[], etc.)

2. **StrEnum**
   - Import from `pipelex.types`:
   ```python
   from pipelex.types import StrEnum
   ```

## BaseModel Standards

- Respect Pydantic v2 standards
- Keep models focused and single-purpose
- Use descriptive field names
- Use type hints for all fields
- Document complex validations
- Use Optional[] for nullable fields
- Use Field(default_factory=...) for mutable defaults

## Factory Pattern

- Use Factory Pattern for object creation when dealing with multiple implementations

## Documentation

1. **Docstring Format**
   ```python
   def process_image(image_path: str, size: Tuple[int, int]) -> bytes:
       """Process and resize an image.
       
       Args:
           image_path: Path to the source image
           size: Tuple of (width, height) for resizing
           
       Returns:
           Processed image as bytes
       """
       pass
   ```

2. **Class Documentation**
   ```python
   class ImageProcessor:
       """Handles image processing operations.
       
       Provides methods for resizing, converting, and optimizing images.
       """
   ```

## Error Handling

1. **Graceful Error Handling**
   - Use try/except blocks with specific exceptions
   - Convert third-party exceptions to custom ones
   ```python
   try:
       from fal_client import AsyncClient as FalAsyncClient
   except ImportError as exc:
       raise MissingDependencyError(
           "fal-client", "fal", 
           "The fal-client SDK is required to use FAL models."
       ) from exc
   ```

## Code Quality Checks

### Linting and Type Checking

Before finalizing a task, run:
```bash
make fix-unused-imports
make check
```

This runs multiple code quality tools:
- Pyright: Static type checking
- Ruff: Fast Python linter  
- Mypy: Static type checker

Always fix any issues reported by these tools before proceeding.

### Running Tests

1. **Quick Test Run** (no LLM/image generation):
   ```bash
   make tp
   ```
   Runs tests with markers: `(dry_runnable or not (inference or llm or imgg or ocr)) and not (needs_output or pipelex_api)`

2. **Specific Tests**:
   ```bash
   make tp TEST=TestClassName
   # or
   make tp TEST=test_function_name
   ```
   Note: Matches names starting with the provided string.

**Important**: Never run `make ti`, `make test-inference`, `make to`, `make test-ocr`, `make tg`, or `make test-imgg` - these use costly inference.

## Pipelines

- All pipeline definitions go in `cocode/pipelex_libraries/pipelines/`
- Always validate pipelines after creation/edit with `make validate`.
  Iterate if there are errors.

## Project Structure

- **Pipelines**: `cocode/pipelex_libraries/pipelines/`
- **Tests**: `tests/` directory
- **Documentation**: `docs/` directory
# Pipeline Guide

- Always first write your "plan" in natural langage, then transcribe it in pipelex.
- You should ALWAYS RUN the terminal command `make validate` when you are writing a `.plx` file. It will ensure the pipe is runnable. If not, iterate.
- Please use POSIX standard for files. (enmpty lines, no trailing whitespaces, etc.)

# Pipeline Structure Guide

## Pipeline File Naming
- Files must be `.plx` for pipelines (Always add an empty line at the end of the file, and do not add trailing whitespaces to PLX files at all)
- Files must be `.py` for structures
- Use descriptive names in `snake_case`

## Pipeline File Structure
A pipeline file has three main sections:
1. Domain statement
2. Concept definitions
3. Pipe definitions

### Domain Statement
```plx
domain = "domain_name"
description = "Description of the domain" # Optional
```
Note: The domain name usually matches the plx filename for single-file domains. For multi-file domains, use the subdirectory name.

### Concept Definitions
```plx
[concept]
ConceptName = "Description of the concept" # Should be the same name as the Structure ClassName you want to output
```

Important Rules:
- Use PascalCase for concept names
- Never use plurals (no "Stories", use "Story")
- Avoid adjectives (no "LargeText", use "Text")
- Don't redefine native concepts (Text, Image, PDF, TextAndImages, Number)
yes 
### Pipe Definitions

## Pipe Base Structure

```plx
[pipe.your_pipe_name]
type = "PipeLLM"
description = "A description of what your pipe does"
inputs = { input_1 = "ConceptName1", input_2 = "ConceptName2" }
output = "ConceptName"
```

DO NOT WRITE:
```plx
[pipe.your_pipe_name]
type = "pipe_sequence"
```

But it should be:

```plx
[pipe.your_pipe_name]
type = "PipeSequence"
description = "....."
```

The pipes will all have at least this base structure. 
- `inputs`: Dictionnary of key behing the variable used in the prompts, and the value behing the ConceptName. It should ALSO LIST THE INPUTS OF THE INTERMEDIATE STEPS (if pipeSequence) or of the conditionnal pipes (if pipeCondition).
So If you have this error:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pipelex/cocode](https://github.com/Pipelex/cocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
