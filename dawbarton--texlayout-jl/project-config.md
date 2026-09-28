---
trigger: always_on
description: This file documents key architectural decisions, invariants, and caveats for
---

# TeXLayout.jl — Architecture and Developer Guide

This file documents key architectural decisions, invariants, and caveats for
agents and human developers working on this codebase.

## Purpose

TeXLayout.jl is a Julia-idiomatic OpenType-aware LaTeX math typesetter.  It is intended
as a drop-in replacement for MathTeXEngine.jl in Makie.jl.  The design reference is
[KaTeX](https://katex.org/); the implementation is not a direct port but follows the same
algorithmic structure where it is sound.

## File structure

```
TeXLayout.jl/
├── src/
│   ├── TeXLayout.jl        # Module entry point; exports and include order
│   ├── math_table.jl       # OpenType MATH table parser + per-font MathTable cache
│   ├── enums.jl            # Internal namespaced enums (EnumX): FontSlot, LayoutMode, Alignment, NodeKind, TokenKind
│   ├── text_styles.jl      # Shared TextAttrs/TextFeatures and nested text-command semantics
│   ├── fonts.jl            # FontFamily, GlyphMetrics, artifact-backed font lookup, font cache
│   ├── style.jl            # TexStyle enum (D/T/S/SS × cramped), style transition helpers
│   ├── lexer.jl            # Tokeniser: LaTeX string → Vector{Token}
│   ├── payloads.jl         # Structured encoders/decoders for data stored in Node.value
│   ├── ast.jl              # Node AST type and AST helper constructors
│   ├── parser.jl           # Recursive-descent parser implementation
│   ├── tables/             # Parser/layout lookup tables kept out of implementation files
│   │   ├── parser_tables.jl
│   │   ├── layout_atoms.jl
│   │   ├── layout_spacing.jl
│   │   └── layout_symbols.jl
│   ├── layout.jl           # Core layout types, shared helpers, recursive dispatch, public layout API
│   ├── layout/             # Feature-specific layout helpers
│   │   ├── constructs.jl
│   │   ├── extensible.jl
│   │   ├── matrix.jl
│   │   └── scripts.jl
│   ├── boxes.jl            # Internal measured box tree + shape pass for composition
│   ├── shaping.jl          # TextShaper interface, MetricShaper, HarfBuzzShaper seam
│   ├── document.jl         # Document AST (Block/Line/Run/TextSpan) + parse_document
│   └── compose.jl          # TeXBox, hconcat, vstack, LayoutOptions, layout_document
├── ext/
│   ├── HarfBuzzExt.jl      # Optional HarfBuzz_jll-backed TextShaper implementation
│   └── MathTeXEngineExt.jl # Makie/MathTeXEngine extension + cached runtime conversion bundle
├── test/
│   ├── runtests.jl         # Top-level testset; includes all test files
│   ├── fixtures/
│   │   └── newcm_math.jl   # Ground-truth constants extracted from NewCMMath-Regular.otf
│   ├── test_math_table.jl  # Tests for MATH table parsing and cache behaviour
│   ├── test_metrics.jl     # Tests for glyph metric lookups
│   ├── test_style.jl       # Tests for style cascade transitions
│   ├── test_lexer.jl       # Tests for the tokeniser
│   ├── test_parser.jl      # Tests for AST structure
│   ├── test_layout.jl      # Tests for layout engine invariants and feature coverage
│   ├── test_katex.jl       # KaTeX-derived test suite (smoke, malformed, nested)
│   ├── test_text.jl        # Text/document layout tests
│   └── test_snapshots.jl   # Layout-equivalence hashes for math and document layout
├── benchmark/
│   ├── Project.toml
│   ├── README.md
│   └── runbenchmarks.jl    # BenchmarkTools harness with baseline comparison support
├── tools/
│   ├── visualise_bitmap.jl        # Rasterise one expression to PNG via FreeType
│   ├── visualise_metrics.jl       # MathTeXEngine-style glyph metric overlay visualiser
│   ├── visualise_metrics_makie.jl # CairoMakie text! + metric overlay visualiser
│   ├── stress_test_content.jl     # Shared test expression definitions (library, not executable)
│   ├── stress_test_freetype.jl    # Render stress-test sheet via FreeType (no Makie required)
│   ├── stress_test_makie.jl       # Render stress-test sheet via CairoMakie
│   ├── stress_test_text.jl        # Render mixed text/math document stress-test sheet
│   ├── stress_test_latex.jl       # Generate .tex stress-test source for xelatex comparison
│   ├── stress_test_suite.jl       # Unified per-case stress PNG generator/packer/comparator
│   ├── stress_test_all.jl         # Compatibility wrapper for stress_test_suite.jl
│   ├── visualise_text.jl          # Render a mixed text/math string to PNG via FreeType
│   └── prepare_font_artifacts.jl  # Build artifact tarballs + draft Artifacts.toml stanzas
├── docs/
│   ├── make.jl             # Documenter.jl build script
│   ├── Project.toml
│   └── src/                # Markdown source pages
├── external/               # Source references (read-only; not part of the package)
│   ├── KaTeX/
│   ├── Makie.jl/
│   └── MathTeXEngine.jl/
├── artifacts/              # Bundled font payloads and extracted font files
├── Artifacts.toml          # Artifact definitions for bundled fonts
├── CHANGELOG.md            # Keep a Changelog format; update [Unreleased] with every change
├── AGENTS.md               # This architecture/developer guide
├── CLAUDE.md               # Symlink to AGENTS.md for compatibility
├── katex_rules.md          # Rule-by-rule implementation notes against KaTeX/TeX
├── notes.md                # Cross-session engineering notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dawbarton/TeXLayout.jl](https://github.com/dawbarton/TeXLayout.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
