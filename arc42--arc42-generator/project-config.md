---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **arc42-generator** project, a Groovy-based build system that converts the arc42 architecture documentation template from its "Golden Master" format (AsciiDoc) into multiple output formats (HTML, PDF, Markdown, DOCX, etc.) in multiple languages.

The actual template content lives in the `arc42-template` git submodule (the "Golden Master"). This generator project transforms that content into various formats for distribution.

## Build Commands

### Initial Setup
```bash
# Initialize and update the arc42-template submodule
git submodule init
git submodule update
cd arc42-template
git checkout master
git pull
cd ..
```

### Full Build Process (Automated)
```bash
./build-arc42.sh
```
This script handles everything: installs pandoc, updates submodules, and runs the full build pipeline.

### Manual Build Steps
```bash
# Full build (all phases)
groovy build.groovy

# Individual phases
groovy build.groovy templates      # Phase 1: Generate templates from golden master
groovy build.groovy convert        # Phases 2-3: Discover + convert templates
groovy build.groovy distribution   # Phase 4: Create distribution ZIP files

# Format-specific build (faster)
groovy build.groovy --format=html  # Build only HTML format
```

### CLI Options
- **Phase selection**: `templates`, `convert`, `distribution`, or `all` (default)
- **Format filter**: `--format=html` (only convert to specified format)
- **Parallel control**: `--parallel=false` (disable parallel execution)

## Architecture

### Build Pipeline Flow
1. **Golden Master** (`arc42-template/` submodule) → Contains source AsciiDoc templates with feature flags
2. **Template Generation** (`lib/Templates.groovy`) → Strips feature flags to create "plain" and "with-help" versions in `build/src_gen/`
3. **Template Discovery** (`lib/Discovery.groovy`) → Scans generated templates and extracts metadata
4. **Format Conversion** (`lib/Converter.groovy`) → Converts AsciiDoc to HTML, Markdown, DOCX, etc. using AsciidoctorJ and Pandoc
5. **Distribution** (`lib/Packager.groovy`) → Packages everything into ZIP files for download

### Core Components

#### `build.groovy` (235 lines)
Main orchestration script that ties everything together. Supports CLI arguments for phase selection and format filtering.

#### `lib/Templates.groovy` (265 lines)
- **Language Auto-Discovery**: Scans `arc42-template/` for language directories matching `/^[A-Z]{2}$/`
- **Feature Flag Removal**: Uses regex patterns to strip `[role="arc42help"]` blocks and `ifdef::arc42help` statements
- **Template Generation**: Creates 18 template variants (9 languages × 2 styles)

**Performance**: Generates templates in ~10s (vs ~30s with Gradle)

#### `lib/Discovery.groovy` (220 lines)
- **Template Scanning**: Discovers all generated templates in `build/src_gen/`
- **Metadata Extraction**: Reads version.properties, counts .adoc files, validates structure
- **Query API**: Find templates by language, style, or both

#### `lib/Converter.groovy` (420 lines)
- **AsciidoctorJ Integration**: Direct HTML and DocBook conversion
- **Pandoc Integration**: Two-step conversion (AsciiDoc → DocBook → target format)
- **Parallel Execution**: Uses GParsPool for true parallel conversion (5-10x faster than Gradle)
- **Supported Formats**: html, asciidoc, docbook, markdown, docx, epub, latex, and more

**Performance**: Converts 18 templates to HTML in ~6s (vs ~45s with Gradle)

#### `lib/Packager.groovy` (205 lines)
- **ZIP Creation**: Packages templates + images into distribution archives
- **Parallel Execution**: Creates all ZIPs concurrently
- **Output**: `arc42-template/dist/*.zip` files ready for distribution

**Performance**: Creates 18 ZIPs in ~0.6s (vs ~15s with Gradle)

### Key Configuration Files
- **buildconfig.groovy**: Defines template styles, output formats, and paths
  - `templateStyles`: `plain` (no help), `with-help` (includes help text)
  - `formats`: 15+ output formats including asciidoc, html, markdown, docx, epub, latex, etc.
  - `goldenMaster`: Path to arc42-template submodule

### Supported Languages
**Auto-discovered**: CZ, DE, EN, ES, FR, IT, NL, PT, RU (9 languages)

The system automatically discovers all language directories in `arc42-template/` that match the pattern `/^[A-Z]{2}$/`. No hardcoding required.

### Format Conversion Strategy
- **AsciiDoc → HTML**: Direct conversion via AsciidoctorJ
- **AsciiDoc → Other formats**: Two-step process
  1. AsciiDoc → DocBook XML (via AsciidoctorJ)
  2. DocBook → Target format (via Pandoc)
- **Multi-page formats**: markdownMP, mkdocsMP, etc. split the template into separate files

### Feature Flag System
The Golden Master uses AsciiDoc role attributes to mark content:
- `[role="arc42help"]` - Help text (explanations, tips)
- `[role="arc42example"]` - Example content (currently unused)
- `lib/Templates.groovy` removes unwanted features using regex to create template variants

### Performance Comparison
**Full HTML Build** (18 templates):
- **Groovy**: 17.4s (template generation + conversion + packaging)
- **Gradle**: ~90s
- **Speedup**: 5.2x faster

**Why Faster**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arc42/arc42-generator](https://github.com/arc42/arc42-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
