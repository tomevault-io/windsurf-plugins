---
trigger: always_on
description: A native Go implementation of the MJML email framework for compiling MJML markup to responsive HTML. 100% feature-complete with all 26 MJML components implemented and thoroughly tested against reference implementation.
---

# gomjml - Native Go MJML Compiler

## Project Overview
A native Go implementation of the MJML email framework for compiling MJML markup to responsive HTML. 100% feature-complete with all 26 MJML components implemented and thoroughly tested against reference implementation.

## Current Development Status
**Branch**: `dev/switch-from-mrml-to-mjml` - Transitioning from MRML-based testing to native reference implementation

The project is currently transitioning away from using MRML (Rust MJML implementation) as the reference for testing and validation. This shift enables:
- **Independence**: No external dependencies for testing and validation
- **Native Control**: Full control over reference implementation and test generation
- **Performance**: Improved development workflow without external tool dependencies
- **Consistency**: Direct alignment with MJML specification rather than third-party interpretation

## Key Architecture
- **CLI Application**: `cmd/gomjml/` with compile and test commands
- **Core Library**: `mjml/` package (importable) with component system
- **Parser**: `parser/` package for XML/AST processing
- **Components**: Individual MJML component implementations in `mjml/components/`

## Build & Test Commands
```bash
# Build CLI (for debugging, always use debug build)
go build -tags debug -o bin/gomjml ./cmd/gomjml

# Build production version (no debug output)
go build -o bin/gomjml ./cmd/gomjml

# Run integration tests (against reference implementation)
./bin/gomjml test

# Run tests with verbose output
./bin/gomjml test -v

# Run tests matching pattern
./bin/gomjml test -pattern "basic"

# Run integration tests with debug build for detailed output
go test -tags debug -v ./mjml -run TestMJMLAgainstExpected

# Run benchmarks
./bench.sh

# Test case comparison with htmlcompare utility
# First build the utility:
go build -o bin/htmlcompare ./cmd/htmlcompare

# Option 1: Run from mjml/testdata/ (auto-detects location):
cd mjml/testdata
../../bin/htmlcompare basic                    # Compare basic.mjml vs basic.html
../../bin/htmlcompare basic --verbose          # Show more diff context

# Option 2: Run from project root (specify testdata directory):
./bin/htmlcompare basic --testdata-dir mjml/testdata         # Compare from root
./bin/htmlcompare mj-button-align --testdata-dir mjml/testdata
./bin/htmlcompare basic -v --testdata-dir mjml/testdata      # Verbose output

# The utility auto-detects project root, builds debug gomjml, and performs semantic HTML comparison

# Compile MJML to HTML
./bin/gomjml compile input.mjml -o output.html

# Compile with debug attributes for troubleshooting
./bin/gomjml compile input.mjml -o output.html --debug

# Debug compile with verbose logging (requires debug build)
./bin/gomjml-debug compile input.mjml -o output.html
```

## Development Guidelines

### Component Interface & Architecture
- **Component Interface**: All components implement `Render(w io.StringWriter) error` and `GetTagName() string`
- **Base Component**: Extend `*BaseComponent` which provides common functionality and attribute handling
- **Testing**: Integration tests validate against reference implementation
- **Performance Focus**: Recent commits show memory optimizations and performance improvements
- **Email Compatibility**: Generates MSO-compatible HTML for Outlook/Gmail/Apple Mail

### Component Implementation Standards

#### HTML Generation
- **Use HTMLTag Builder**: Use `html.NewHTMLTag()` for generating HTML elements instead of string concatenation
- **StringWriter Pattern**: All internal rendering methods must use `io.StringWriter` interface for performance
- **Constants Usage**: Use constants from `mjml/constants` package instead of hardcoded strings:
  - CSS properties: `constants.CSSFontSize`, `constants.CSSPadding`, etc.
  - HTML attributes: `constants.AttrClass`, `constants.AttrCellSpacing`, etc.
  - MJML attributes: `constants.MJMLFontFamily`, `constants.MJMLPadding`, etc.
  - Common values: `constants.VAlignMiddle`, `constants.AlignCenter`, etc.
  - Language/Direction: `constants.LangUndetermined`, `constants.DirAuto`, etc.
  - **🚨 Code Review Check**: Always look for magic strings that should be constants!

#### Code Structure
- **Font Tracking**: Always call `c.TrackFontFamily(value)` when processing font-family attributes
- **Attribute Handling**: 
  - Use `c.GetAttributeWithDefault(c, name)` directly instead of creating wrapper `getAttribute` methods
  - Only create custom attribute methods when implementing specific inheritance patterns (like accordion element)
  - For explicit-only attributes like font-family, use `c.Node.GetAttribute(name)` directly
- **Individual Padding**: Support individual padding properties (`padding-top`, `padding-bottom`, etc.) alongside general `padding`
- **MSO Compatibility**: Use MSO conditional comments for Outlook-specific elements: `<!--[if !mso | IE]><!--> ... <!--<![endif]-->`
- **CSS Class Support**: ALL components MUST support the `css-class` attribute using `c.BuildClassAttribute()` on their main container element

#### Example Pattern
```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [preslavrachev/gomjml](https://github.com/preslavrachev/gomjml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
