---
trigger: always_on
description: EZpanso is a modern GUI application for managing Espanso text expansion snippets built with PyQt6 and PyYAML. The application provides a user-friendly interface for editing, creating, and managing YAML-based text expansion files.
---

# GitHub Copilot Instructions for EZpanso

## Project Overview

EZpanso is a modern GUI application for managing Espanso text expansion snippets built with PyQt6 and PyYAML. The application provides a user-friendly interface for editing, creating, and managing YAML-based text expansion files.

## Python Coding Standards

This project follows **PEP 8** (Python Enhancement Proposal 8) as the primary coding standard, with the following specific guidelines:

### Code Style
- **Line Length**: Maximum 88 characters (Black formatter compatible)
- **Indentation**: 4 spaces (no tabs)
- **Quotes**: Double quotes for strings, single quotes for character literals
- **Import Organization**: Follow PEP 8 import order (standard library, third-party, local)
- **Naming Conventions**:
  - Classes: PascalCase (`EZpanso`, `FileData`)
  - Functions/Methods: snake_case (`_setup_ui`, `_load_yaml_files`)
  - Constants: UPPER_SNAKE_CASE (`MAX_UNDO_STEPS`)
  - Private methods: Leading underscore (`_internal_method`)

### Type Hints
- Use type hints for all function signatures and class attributes
- Import types from `typing` module when needed
- Use `Optional[T]` for nullable types
- Define type aliases for complex types (e.g., `FileData = Dict[str, List[Dict[str, Any]]]`)

### Documentation
- Use docstrings for all public methods and classes
- Follow Google-style docstrings format
- Include type information in docstrings when helpful
- Document complex algorithms and business logic
- Update `README.md` and `CHANGELOG.md` for significant changes
- Use `Context7` to reference PyQt6 and PyYAML documentation for specific methods and classes

### Error Handling
- Use specific exception types rather than bare `except:`
- Handle PyQt6-specific exceptions appropriately
- Provide meaningful error messages to users
- Log errors for debugging purposes

## Architecture Guidelines

### Single Responsibility Principle
- Each method should have a single, well-defined purpose
- Separate UI logic from business logic where possible
- Keep data manipulation separate from presentation

### PyQt6 Best Practices
- Use signals and slots for communication between components
- Implement proper event handling
- Follow Qt's parent-child object model
- Use Qt's built-in data types and structures when appropriate
- Always use `yaml.safe_load()` instead of `yaml.load()` for security
- Block signals during bulk UI updates to prevent cascading events
- Use `QSettings` for persistent application settings
- Implement proper resource cleanup in `closeEvent()`

### File Structure
- Keep the monolithic structure for simplicity
- Group related methods together
- Use clear section comments to organize code
- Maintain consistent indentation and spacing

## Dependencies and External Libraries

### Core Dependencies
- **PyQt6**: GUI framework - refer to official documentation for best practices
- **PyYAML**: YAML parsing and generation - ensure proper handling of special characters
- **Python Standard Library**: Use built-in modules when possible

### Development Dependencies
- **PyInstaller**: For building standalone applications
- **pytest**: For unit testing

## Code Quality Guidelines

### Testing
- Test files are located at `tests/`
- Write unit tests for critical business logic
- Mock external dependencies (file system, settings)
- Test edge cases and error conditions
- Maintain test coverage for core functionality

### Performance
- Use efficient data structures (lists, dicts) appropriately
- Avoid unnecessary file I/O operations
- Cache frequently accessed data
- Optimize UI updates to prevent blocking

### Security
- Validate user input before processing
- Sanitize file paths to prevent directory traversal
- Handle YAML parsing safely to prevent code injection
- Backup user data before making changes

## Specific Guidelines for EZpanso

### YAML Handling
- **Security**: Always use `yaml.safe_load()` instead of `yaml.load()` to prevent code injection
- Preserve original file structure and comments when possible
- Handle special characters (newlines, tabs) properly using escape sequences
- Validate YAML syntax before saving to prevent corruption
- Provide clear error messages for malformed files
- Use UTF-8 encoding for all file operations
- Implement atomic file writes (write to temp file, then rename) for data safety

### GUI Design
- Maintain consistent styling across components
- Use appropriate widget types for data input
- Implement keyboard shortcuts for common actions
- Provide visual feedback for user actions

### Data Management
- Track file modifications accurately
- Implement undo/redo functionality
- Handle concurrent file access gracefully
- Validate data integrity before operations

### Cross-Platform Compatibility
- Use cross-platform file path handling
- Test on multiple operating systems
- Handle platform-specific differences gracefully
- Use appropriate keyboard shortcuts for each platform

## Code Review Checklist

When reviewing code changes:

1. **Functionality**: Does the code work as intended?
2. **Style**: Does it follow PEP 8 and project conventions?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luklongman/EZpanso](https://github.com/luklongman/EZpanso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
