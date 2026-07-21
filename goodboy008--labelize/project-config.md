---
trigger: always_on
description: Labelize is a Rust engine that parses ZPL (Zebra Programming Language) and EPL (Eltron Programming Language) label data and renders it to PNG or PDF. It runs as a CLI tool, HTTP microservice, or library.
---

# Labelize — Project Guidelines

## Overview

Labelize is a Rust engine that parses ZPL (Zebra Programming Language) and EPL (Eltron Programming Language) label data and renders it to PNG or PDF. It runs as a CLI tool, HTTP microservice, or library.

## Architecture

```
ZPL/EPL input (bytes)
  → Parser (src/parsers/)        — splits commands, tracks VirtualPrinter state
  → Elements (src/elements/)     — typed structs: Text, Barcode128, GraphicBox, etc.
  → LabelInfo                    — { elements, print_width, inverted }
  → Renderer (src/drawers/)      — rasterizes to RgbaImage canvas
  → Encoder (src/images/)        — PNG (monochrome 1-bit) or PDF
```

### Key Modules

| Module | Purpose |
|--------|---------|
| `src/parsers/zpl_parser.rs` | ZPL command parser with VirtualPrinter state machine |
| `src/parsers/epl_parser.rs` | EPL command parser |
| `src/elements/` | 30+ element structs (text, barcodes, graphics, stored formats) |
| `src/drawers/renderer.rs` | Image canvas creation, element drawing, compositing |
| `src/barcodes/` | Barcode encoders: Code128, Code39, EAN-13, 2of5, PDF417, Aztec, DataMatrix, QR, MaxiCode |
| `src/encodings/` | Character set encoding maps |
| `src/assets/fonts/` | Embedded fonts: Helvetica Bold Condensed, DejaVu Sans Mono, ZPL GS |

## ZPL Official Reference

The authoritative specification for all ZPL commands is the Zebra Programming Guide:
https://docs.zebra.com/us/en/printers/software/zpl-pg/c-zpl-zpl-commands.html

Individual command docs follow the pattern:
`https://docs.zebra.com/us/en/printers/software/zpl-pg/c-zpl-zpl-commands/r-zpl-<cmd>.html`

**Always consult this reference when:**
- Implementing or fixing any ZPL command parser logic
- Rendered output doesn't match expected behavior
- Parameter defaults, ranges, or interactions are unclear
- Adding support for new ZPL commands

## Supported ZPL Commands

**Label control:** `^XA`, `^XZ`, `^LH`, `^LR`, `^PO`, `^PW`, `^CI`, `^CF`
**Text/Font:** `^A`, `^FO`, `^FT`, `^FW`, `^FD`, `^FS`, `^FB`, `^FV`, `^FN`, `^FR`, `^FH`
**Barcodes:** `^BC` (Code128), `^BE` (EAN-13), `^B2` (2of5), `^B3` (Code39), `^B7` (PDF417), `^BO` (Aztec), `^BX` (DataMatrix), `^BQ` (QR), `^BD` (MaxiCode), `^BY` (defaults)
**Graphics:** `^GB`, `^GC`, `^GD`, `^GF`, `^GS`
**Storage:** `~DG`, `^IL`, `^XG`, `^DF`, `^XF`

## Build & Test

```bash
# Build
cargo build

# Run all tests
cargo test

# Run all e2e tests (golden comparison against Labelary reference images)
cargo test --test 'e2e_*'

# Run all unit tests
cargo test --test 'unit_*'

# Run a specific test file
cargo test --test unit_renderer

# Run e2e golden tests with output
cargo test --test e2e_golden -- --nocapture

# Convert a label
cargo run -- convert testdata/amazon.zpl

# Bootstrap Labelary reference PNGs for any new ZPL files (requires network)
cargo test --test e2e_labelary bootstrap_golden_pngs -- --ignored --nocapture

# Regenerate all diff images and diff reports after any rendering change
cargo test --test e2e_diff_report -- --nocapture
```

## Testing Conventions

- **Golden tests** compare rendered PNGs pixel-by-pixel against reference images from Labelary
- Reference images live in `testdata/labels/*.png` and `testdata/unit/*.png`; diff outputs go to `testdata/diffs/`
- Each label has a tolerance threshold documented in `docs/DIFF_THRESHOLDS.md`
- After any rendering change, run `cargo test --test e2e_golden` to verify diff percentages stay within tolerance
- Test files follow naming convention: `tests/e2e_*.rs` for e2e tests, `tests/unit_*.rs` for unit tests
- Shared test utilities are in `tests/common/` (not compiled as tests)

### Testdata Directory Layout

| Directory | Canvas size | Labelary params | Purpose |
|-----------|-------------|------------------|---------|
| `testdata/labels/` | **813×1626 px** | 101.625mm × 203.25mm | Carrier/real-world labels |
| `testdata/unit/` | **812×1624 px** | 101.5mm × 203.0mm | Synthetic/unit fixture labels |

These sizes come from `render_helpers::default_options()` and `render_helpers::unit_options()` respectively.

### Adding a New ZPL Test File

1. Drop `<name>.zpl` into `testdata/unit/` (synthetic) or `testdata/labels/` (real-world carrier label)
2. Fetch its Labelary reference PNG (requires network):
   ```bash
   cargo test --test e2e_labelary bootstrap_golden_pngs -- --ignored --nocapture
   ```
   This scans both directories and fetches a PNG **only** for ZPL files that don't already have one. Existing PNGs are never overwritten.
3. Generate the diff comparison image:
   ```bash
   cargo test --test e2e_diff_report -- --nocapture
   ```
4. Commit both the `.zpl` and `.png` files, plus the updated files in `testdata/diffs/`.

### Rendering Change Workflow

When you modify any rendering-related logic (parsers, elements, drawers, barcodes, fonts, encoders), you **must**:

1. Run `cargo test --test e2e_diff_report -- --nocapture` to regenerate all diff images and both diff reports:
   - `testdata/diffs/diff_report_labels.txt` — carrier/real-world labels (813×1626)
   - `testdata/diffs/diff_report_unit.txt` — unit/synthetic tests (812×1624)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GOODBOY008/labelize](https://github.com/GOODBOY008/labelize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
