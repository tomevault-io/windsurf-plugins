---
trigger: always_on
description: **RefreshCSS** is a pure Python library that removes unused CSS selectors (classes, ids, and elements) from CSS files based on what's actually used in HTML templates. It's designed to work seamlessly with Django/Jinja-style templates and can be integrated into build processes.
---

# AGENTS.md - RefreshCSS Library Analysis

## Overview

**RefreshCSS** is a pure Python library that removes unused CSS selectors (classes, ids, and elements) from CSS files based on what's actually used in HTML templates. It's designed to work seamlessly with Django/Jinja-style templates and can be integrated into build processes.

**Version**: 0.5.1  
**License**: MIT  
**Python**: 3.10+

## Core Purpose

RefreshCSS performs CSS "tree-shaking" or "purging" - it analyzes HTML templates to find which CSS selectors are actually being used, then outputs a cleaned CSS file containing only those selectors. This reduces CSS file size and improves performance.

## Architecture

### High-Level Flow

1. **Parse HTML** → Extract all classes, ids, and elements from HTML templates.
   - Uses **JustHTML** for static HTML structure.
   - Uses **Aggressive Regex Extraction** for classes and IDs inside Django/Jinja template tags (limited to `class` and `id` attributes).
2. **Parse CSS** → Extract all rules and selectors from CSS files using **tinycss2**.
3. **Compare** → Match CSS selectors against HTML usage using efficient set operations.
4. **Output** → Return cleaned CSS containing only used selectors, preserving comments and at-rules.

### Core Components

#### 1. HTML Parsing (`src/refreshcss/html/`)

**`File` class** (`html/file.py`):
- Parses individual HTML files using a combination of **JustHTML** and targeted regex.
- Extracts three types of selectors:
  - **Classes**: 
    - **Static**: Extracted via `JustHTML` from the DOM (template tags are replaced with spaces to avoid "smashing" adjacent classes).
    - **Aggressive**: Extracted from Django/Jinja template tags (e.g., `{{ ... }}`, `{% ... %}`) found strictly within `class="..."` HTML attributes.
  - **IDs**: 
    - **Static**: Extracted via `JustHTML`.
    - **Aggressive**: Extracted from template tags found strictly within `id="..."` HTML attributes.
  - **Elements**: Extracted via `JustHTML` tag name traversal.
- Uses cached properties for performance.

**Key Regex Patterns**:
- `CLASS_ATTR_RE`: Matches `class="..."` attributes, handling nested template tags and quotes.
- `ID_ATTR_RE`: Matches `id="..."` attributes.
- `TEMPLATE_TAG_RE`: Matches `{% ... %}` and `{{ ... }}`.
- `CLASS_IN_TEMPLATE_RE`: Matches quoted strings within template tags that look like CSS classes/IDs.

**`Site` classes** (`html/site.py`):
- **`Site`**: Abstract base class that aggregates classes, ids, and elements from multiple files
- **`DjangoSite`**: Automatically discovers Django template directories and uses Django's default charset
- **`PathSite`**: Accepts explicit file paths, supports recursive directory scanning
- **`UrlSite`**: Placeholder for future URL-based crawling (not yet implemented)

#### 2. CSS Parsing (`src/refreshcss/css/`)

**`parser.py`** (`css/parser.py`):
- Uses **tinycss2** to parse CSS into an AST-like stream of tokens and rules.
- **Key algorithm**:
  1. `tinycss2.parse_stylesheet` converts CSS to rules while preserving comments.
  2. For each **QualifiedRule** (regular CSS rule):
     - `_rule_is_kept` performs a single-pass scan over the selector prelude.
     - Short-circuits as soon as any selector in a comma-separated list is deemed "used".
     - Validates against `site.classes`, `site.ids`, and `site.elements`.
  3. For each **AtRule** (e.g., `@media`, `@container`, `@supports`):
     - Recursively filters nested rules.
     - Removes at-rules that become empty after filtering.
     - Preserves non-nested at-rules like `@font-face` or `@keyframes`.
  4. Serializes the kept rules back to CSS text.

**Special Cases Handled**:
- **Modern CSS**: Full support for nesting, container queries, and complex selectors via `tinycss2`.
- **Comments**: Preserved by default as they are included in the `tinycss2` rule list.
- **Universal selector (`*`)**: Always preserved.
- **Pseudo-elements/classes**: Correctly ignored during base-selector matching.

#### 3. Main Interface (`refreshcss/main.py`)

**`RefreshCSS` class**:
- Tie common logic together.
- The `clean()` method accepts CSS text and returns cleaned CSS.

#### 4. Utilities (`refreshcss/utils/`)

**`path.py`**:
- `read_text()`: Reads files with optional encoding detection using `charset-normalizer`.

**`string.py`**:
- `finditer()`: Yields start/end indices for substring matches
- `remove_string_at()`: Removes substring by index range

### Integration Points

#### 1. Command-Line Interface (`src/refreshcss/cli.py`)

Uses Click framework to provide a CLI:
```bash
refreshcss [OPTIONS] CSS HTML...

Options:
  -o, --output FILENAME  # Write to file instead of stdout
  -R, -r, --recursive    # Recursively search subdirectories
  --encoding TEXT        # Character encoding
```

**Example**:
```bash
refreshcss styles.css templates/ -r -o cleaned.css
```

#### 2. Django Compressor Filter (`src/refreshcss/filters.py`)

**`RefreshCSSFilter`**:
- Integrates with `django-compressor` as a filter.
- Caches the parsed `DjangoSite` object for performance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamghill) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
