---
trigger: always_on
description: Airlift is a Python command-line tool for uploading CSV/JSON data with attachments to Airtable. The project uses a modular architecture with clear separation of concerns across data processing, API integration, and file handling components.
---

# Airlift Development Rules and Guidelines

## Project Context

Airlift is a Python command-line tool for uploading CSV/JSON data with attachments to Airtable. The project uses a modular architecture with clear separation of concerns across data processing, API integration, and file handling components.

## Quick Reference Commands

### Building and Testing
```bash
# Build the project using the ephemeral build system (recommended)
./scripts/local-test-build.sh

# Test the built binary
./test-build/airlift --help

# Clean build environment
./scripts/local-test-build.sh --clean

# Build with dependency updates
./scripts/local-test-build.sh --update-deps

# Run comprehensive tests (no API tokens required)
./scripts/local-test-build.sh --comprehensive-test

# Dev CLI (after --comprehensive-test or full build)
.build/venv/bin/airlift --help

# Prefer the binary for release parity
./test-build/airlift --help
```

### Dependency Management
```bash
# Regenerate poetry.lock (use ephemeral Python 3.14.5 in .build/)
./scripts/local-test-build.sh --lock-only

# Update one package at a time (recommended)
./scripts/local-test-build.sh --lock-only --update <package>
./scripts/local-test-build.sh --comprehensive-test

# Lock + full build for a package
./scripts/local-test-build.sh --update <package>

# Bump all packages within ^ ranges (use sparingly)
./scripts/local-test-build.sh --update-deps

# Show outdated packages
./scripts/local-test-build.sh --show-outdated
```

After editing `pyproject.toml`, use the build script—not bare `poetry lock` on system Python.

## Code Organization

### Module Structure
- Keep each module focused on a single responsibility
- Use clear, descriptive module names that reflect their purpose
- Maintain consistent import patterns across modules
- Avoid circular dependencies between modules

### File Naming Conventions
- Use snake_case for all Python files and functions
- Use descriptive names that clearly indicate functionality
- Follow the existing naming pattern: `airlift_*.py` for core modules

## Coding Standards

### Python Style
- Follow PEP 8 style guidelines strictly
- Use type hints for all function parameters and return values
- Implement comprehensive docstrings for public functions and classes
- Use logging instead of print statements for all output
- Keep line length under 88 characters (Black formatter standard)

### Error Handling
- Use custom exception classes from `utils_exceptions.py`
- Implement proper exception chaining with `raise ... from`
- Provide meaningful error messages to end users
- Log detailed error information for debugging
- Handle both critical and non-critical errors appropriately

### Data Processing
- Validate input data before processing
- Use UTF-8 encoding for all file operations
- Handle missing or malformed data gracefully
- Implement proper data type conversion and validation

## API Integration Patterns

### Airtable API
- Use the `new_client` class for all Airtable operations
- Implement proper authentication with Bearer tokens
- Handle API rate limits and errors gracefully
- Use structured JSON payloads for data uploads
- Validate responses and handle error codes appropriately
- Use pyairtable 3.x APIs for field creation and schema management
- Use batch operations for delete (10 records per API call)

### Dropbox API
- Use the `dropbox_client` class for file operations
- Implement OAuth2 flow for authentication with explicit scopes
- Handle refresh token management properly
- Create organized folder structures for uploads
- Generate proper sharing URLs for attachments
- Use Dropbox SDK 12.x for latest API features and security

## CLI Development

### Argument Parsing
- Use the existing `cli_args.py` structure for argument definitions
- Group related arguments logically (general, dropbox, column, validation, database options)
- Provide clear help text for all options
- Implement proper validation for required arguments in `_validate_required_args()`
- Use appropriate data types for argument values

### CLI Arguments Structure
```python
schema: ArgSchema = {
    "POSITIONAL": { ... },
    "general_options": { ... },
    "dropbox options": { ... },
    "column_options": { ... },
    "custom application options": { ... },
    "validation_options": { ... },
    "database_options": { ... },  # --delete-all-database-entries, --empty-dropbox-folder
}
```

### User Experience
- Provide clear progress indicators for long operations (tqdm)
- Use consistent logging levels (INFO, WARNING, ERROR, DEBUG)
- Implement verbose mode for detailed debugging output
- Handle user interruptions gracefully (Ctrl+C)
- Show helpful error messages for missing required arguments

## Performance Considerations

### Concurrency
- Use ThreadPoolExecutor for parallel upload operations
- Implement configurable worker thread counts (default: 5)
- Avoid blocking operations in worker threads
- Use proper queue management for data distribution

### Memory Management
- Process data in chunks for large files
- Avoid loading entire datasets into memory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAcharya/Airlift](https://github.com/TheAcharya/Airlift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
