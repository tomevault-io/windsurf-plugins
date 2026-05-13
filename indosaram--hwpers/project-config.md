---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`hwpers` is a Rust library for parsing Korean Hangul Word Processor (HWP) 5.0 files with full layout rendering support. As of v0.3.0, it includes both **reader** (parsing) and **writer** (creation) capabilities.

## Development Commands

### Building
```bash
cargo build              # Debug build
cargo build --release    # Release build (optimized)
cargo check              # Fast syntax/type check without producing binary
```

### Testing
```bash
cargo test                        # Run all tests
cargo test <test_name>           # Run specific test (substring match)
cargo test --release             # Run tests in release mode (faster execution)
cargo test -- --nocapture        # Show println! output from tests
cargo test --test <test_file>    # Run specific test file (e.g., integration_test)
```

### Code Quality
```bash
cargo clippy             # Linter - catches common mistakes and suggests improvements
cargo fmt                # Format code according to Rust style guidelines
cargo doc                # Generate and open documentation
cargo doc --open         # Generate docs and open in browser
```

### Running the CLI Tool
```bash
cargo run --bin hwp_info -- <file.hwp>    # Inspect HWP file structure
cargo install --path .                    # Install hwp_info locally
hwp_info <file.hwp>                      # If installed
```

## Architecture

### Core Components

The codebase is organized into distinct layers:

1. **Reader Layer** (`src/reader/`)
   - `cfb.rs` - Compound File Binary format handler
   - `stream.rs` - Stream abstraction for reading CFB streams
   - Entry point: `HwpReader::from_file()` or `HwpReader::from_bytes()`

2. **Parser Layer** (`src/parser/`)
   - `header.rs` - File header and metadata parsing
   - `doc_info.rs` - Document properties, styles, fonts, formatting tables
   - `body_text.rs` - Content sections (sections → paragraphs → text)
   - `record.rs` - Low-level record parsing
   - Parsers operate on uncompressed streams and build the model

3. **Model Layer** (`src/model/`)
   - `document.rs` - Root `HwpDocument` struct
   - `paragraph.rs` - Text paragraphs with formatting
   - `char_shape.rs` - Character formatting (font, size, style)
   - `para_shape.rs` - Paragraph formatting (alignment, indent)
   - `control.rs` - Embedded objects (tables, images, etc.)
   - `hyperlink.rs`, `header_footer.rs`, `page_layout.rs`, etc.
   - The model is a complete in-memory representation of the HWP document

4. **Render Layer** (`src/render/`)
   - `layout.rs` - Layout engine that reconstructs visual layout
   - `renderer.rs` - SVG renderer
   - Converts document model + layout data → visual representation

5. **Writer Layer** (`src/writer/`) - **v0.3.0+**
   - `mod.rs` - `HwpWriter` API for creating HWP files
   - `serializer.rs` - Converts model to CFB format
   - `style.rs` - Style definitions and formatting
   - **Status**: Early development, many features incomplete (see README limitations)

6. **Utils Layer** (`src/utils/`)
   - `compression.rs` - zlib compression/decompression
   - `encoding.rs` - UTF-16LE and other encodings

### Document Structure Hierarchy

```
HwpDocument
├── FileHeader              # Format version, compression flags
├── DocInfo                 # Document-wide properties
│   ├── Styles              # Style definitions
│   ├── CharShapes          # Character formatting table
│   ├── ParaShapes          # Paragraph formatting table
│   ├── BorderFills         # Border/fill definitions
│   └── FaceNames           # Font definitions
└── BodyTexts               # Content sections
    └── Sections            # Page sections
        └── Paragraphs      # Text paragraphs
            ├── ParaText    # Actual text content
            ├── CharShapes  # Character runs (position → style)
            └── Controls    # Embedded objects (tables, images)
```

### Key Design Patterns

**Zero-copy parsing**: Where possible, parsers avoid allocating new strings and instead reference the original byte buffer.

**Formatting tables**: Character and paragraph formatting are stored in document-wide tables (in `DocInfo`), and paragraphs reference them by ID. This is why you see `char_shape_id` and `para_shape_id` fields.

**Control objects**: Embedded content (tables, images, text boxes) are represented as "controls" with a control header defining type and properties.

**Layout reconstruction**: When layout data is present (line segments, character positions), the render layer can reconstruct pixel-perfect positioning. Otherwise, it falls back to estimated layout.

## Testing Strategy

Tests are organized in the `tests/` directory as integration tests:

- `integration_test.rs` - Basic parsing and text extraction
- `writer_test.rs` - Document creation
- `roundtrip_test.rs` - Write → Read verification
- `hyperlink_test.rs`, `table_test.rs`, `image_test.rs`, etc. - Feature-specific tests
- `serialization_test.rs` - Low-level format serialization

Test files are stored in `test-files/` directory (not committed to git).

## Writer Limitations (v0.3.0)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Indosaram/hwpers](https://github.com/Indosaram/hwpers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
