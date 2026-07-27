---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference

```bash
# Using justfile (recommended)
just help          # Show all available commands
just test          # Run tests
just format        # Format code
just workflow      # Full dev cycle: install, format, test

# Using composer directly
composer test      # Run tests
composer format    # Format code
composer serve     # Start dev server
```

## Development Commands

### Using Justfile (Preferred)

This project uses [just](https://just.systems/) as a command runner:

```bash
# Setup
just install              # Install PHP dependencies
just env                  # Copy .env.example to .env

# Testing
just test                 # Run all tests
just coverage             # Run tests with coverage (uses herd if available)
just test-unit            # Run unit tests only
just test-feature         # Run feature tests only
just test-integration     # Run integration tests only
just test-file <path>     # Run a specific test file
just test-filter <name>   # Run tests matching a filter
just test-laravel         # Test Laravel 10/11/12 compatibility

# Code Quality
just format               # Format code with Laravel Pint
just format-check         # Check formatting without changes
just analyse              # Run PHPStan (if configured)
just lint                 # Format code
just check                # Format + test

# Development
just build                # Build the package
just serve                # Start development server
just update               # Update composer dependencies

# Workflows
just workflow             # Full cycle: install, format, test
just quick                # Quick check: format, test
just pre-commit           # Pre-commit check
just ci                   # Simulate CI: format-check, test

# Cleanup
just clean                # Remove generated files
just fresh                # Clean + install
```

### Using Composer

```bash
composer test              # Run the full Pest test suite
composer test-coverage     # Run tests with code coverage
composer format            # Format code using Laravel Pint
composer lint              # Alias for format
composer analyse           # Run PHPStan (requires phpstan.neon)
composer serve             # Build and serve via Testbench
composer build             # Build the package
```

## High-Level Architecture

This is a Laravel package that provides AI-powered data extraction using OpenAI's API. The architecture follows these key patterns:

### Core Components

1. **Engine (`src/Engine.php`)**: Orchestrates all AI interactions with OpenAI. Handles different model types (completion, chat, vision, JSON mode) through a unified `run()` method. Model selection determines payload format (vision models get image URLs, JSON-mode models get response_format settings).

2. **ExtractorManager (`src/ExtractorManager.php`)**: Main entry point via the `Extractor` facade. Manages extractor registration (`extend()`), resolution, and execution. Delegates actual AI calls to the Engine.

3. **Extractor Pipeline (`src/Extraction/Extractor.php`)**: Base class for all extractors using a template method pattern:
    - `preprocess($input)` → Transform input before AI processing (via registered preprocessors)
    - `prompt($input)` → Generate AI prompt using Blade templates
    - Engine execution → Send to OpenAI API
    - `process($response)` → Transform AI response into desired format (via registered processors)

4. **Text Loading System (`src/Text/`)**: Factory pattern for loading various file formats:
    - `Factory` class provides `create(type)` and convenience methods (`pdf()`, `html()`, etc.)
    - `TextLoader` interface implemented by format-specific loaders
    - Supports PDF, Word, RTF, HTML, images, and web content
    - AWS Textract integration for OCR functionality

### Extension Points

- **Custom Extractors**: Extend `Extractor` class, override `prompt()` method, optionally use `HasValidation` or `HasDto` traits
- **Custom Text Loaders**: Implement `TextLoader` interface, register in `Factory::create()` match statement
- **Custom Prompts**: Publish and modify Blade templates in `resources/prompts/`
- **Processors/Preprocessors**: Register via `registerProcessor()` and `registerPreprocessor()` with priority ordering

### Design Patterns Used

| Pattern               | Implementation                                              | Location                       |
| --------------------- | ----------------------------------------------------------- | ------------------------------ |
| **Strategy**          | Different extraction strategies via Extractor subclasses    | `src/Extraction/Builtins/`     |
| **Factory**           | TextLoader factory creates appropriate loaders by type/MIME | `src/Text/Factory.php`         |
| **Template Method**   | Extractor base class defines extraction workflow hooks      | `src/Extraction/Extractor.php` |
| **Facade**            | Laravel facades for convenient static access                | `src/Facades/`                 |
| **Pipeline**          | Composable processors/preprocessors with priority ordering  | `src/Extraction/Extractor.php` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HelgeSverre/extractor](https://github.com/HelgeSverre/extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
