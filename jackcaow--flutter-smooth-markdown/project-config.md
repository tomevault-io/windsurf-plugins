---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Flutter Smooth Markdown** is a high-performance Flutter markdown renderer with syntax highlighting, LaTeX math, tables, footnotes, SVG images, and real-time streaming support. The package provides a custom AST-based parser and extensible widget builder system.

- **Version**: 0.2.0
- **Minimum SDK**: Dart >=3.0.0, Flutter >=3.0.0
- **Repository**: https://github.com/JackCaow/flutter-smooth-markdown

## Development Commands

### Testing
```bash
# Run all tests
flutter test

# Run specific test file
flutter test test/parser/markdown_parser_test.dart

# Run tests with coverage
flutter test --coverage

# Run tests for a specific pattern
flutter test --name="header"
```

### Build and Analyze
```bash
# Analyze code quality
flutter analyze

# Get dependencies
flutter pub get

# Run example app (important for visual testing)
cd example && flutter run

# Check available devices
flutter devices
```

### Publishing
```bash
# Dry run publish (no actual upload)
flutter pub publish --dry-run

# Publish to pub.dev (requires authentication)
flutter pub publish
```

## Architecture

The codebase uses a **layered architecture** with clear separation of concerns:

### 1. Parser Layer (`lib/src/parser/`)
- **MarkdownParser**: Main parser that orchestrates block and inline parsing
- **BlockParser**: Handles block-level elements (headers, paragraphs, lists, code blocks, tables, blockquotes, horizontal rules)
- **InlineParser**: Handles inline elements (bold, italic, links, images, inline code, strikethrough, math formulas)
- **AST**: Abstract Syntax Tree nodes representing parsed markdown structure (`lib/src/parser/ast/markdown_node.dart`)

**Parsing Flow**:
```
Raw Markdown → BlockParser → Block AST Nodes → InlineParser processes text content → Final AST
```

### 2. Renderer Layer (`lib/src/renderer/`)
- **MarkdownRenderer**: Main renderer with builder registry system
- **WidgetBuilder**: Base interface for all element builders
- **BuilderRegistry**: Maps node types to builders
- **Builders** (`lib/src/renderer/builders/`): Individual builders for each markdown element type
  - Standard builders: text, header, paragraph, code_block, list, blockquote, link, image, table, etc.
  - Enhanced builders: enhanced_code_block, enhanced_blockquote, enhanced_link, enhanced_header (with rich UI features)
  - Math builders: inline_math, block_math (LaTeX rendering)
  - Footnote builders: footnote_reference, footnote_definition

**Rendering Flow**:
```
AST Nodes → MarkdownRenderer → BuilderRegistry lookup → Specific WidgetBuilder → Flutter Widget Tree
```

### 3. Widget Layer (`lib/widgets/`)
- **SmoothMarkdown**: Main widget for static markdown rendering
- **StreamMarkdown**: Widget for real-time streaming markdown (incremental rendering)

### 4. Configuration Layer (`lib/src/config/`)
- **MarkdownStyleSheet**: Theming system with presets (light, dark, GitHub, VS Code)
- **MarkdownConfig**: Feature toggles and parser configuration

## Key Design Patterns

### Builder Pattern with Registry
The renderer uses a builder registry system that allows customization:

```dart
final renderer = MarkdownRenderer(styleSheet: MarkdownStyleSheet.light());

// Register custom builders
renderer.builderRegistry
  ..register('code_block', const EnhancedCodeBlockBuilder())
  ..register('link', const CustomLinkBuilder());
```

### Immutable AST Nodes
All AST nodes are immutable with `copyWith()` methods for modifications. This ensures thread safety and enables efficient caching.

### Extensibility Points
1. **Custom Builders**: Implement `MarkdownWidgetBuilder` interface
2. **Custom Themes**: Extend `MarkdownStyleSheet` with `copyWith()`
3. **Parser Extensions**: Modify block/inline parser behavior

## Test Structure

Tests are organized by layer:
- `test/parser/`: Parser tests (ast_test.dart, block_parser_test.dart, inline_parser_test.dart, markdown_parser_test.dart)
- `test/renderer/`: Renderer tests (widget_builder_test.dart, smooth_markdown_test.dart, table_builder_test.dart)
- `test/integration/`: Integration tests (table_integration_test.dart)

**Running specific test suites**:
```bash
flutter test test/parser/          # All parser tests
flutter test test/renderer/        # All renderer tests
flutter test test/integration/     # Integration tests
```

## Code Quality Standards

The project uses strict linting with `flutter_lints` and additional rules:
- **Strict mode enabled**: `strict-casts`, `strict-inference`, `strict-raw-types`
- **130+ lint rules** enforced (see `analysis_options.yaml`)
- **All public APIs require documentation comments** using `///`
- **Type annotations required** for public APIs

**Before committing**:
```bash
flutter analyze    # Must pass with no issues
flutter test       # All tests must pass
```

## Key Dependencies

**Production**:
- `flutter_highlight` (^0.7.0): Syntax highlighting for code blocks
- `cached_network_image` (^3.3.0): Image caching and network loading
- `url_launcher` (^6.2.0): Handle URL taps
- `flutter_math_fork` (^0.7.2): LaTeX math formula rendering
- `flutter_svg` (^2.0.10+1): SVG image support

**Development**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JackCaow/flutter-smooth-markdown](https://github.com/JackCaow/flutter-smooth-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
