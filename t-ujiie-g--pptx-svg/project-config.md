---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build Wasm (output: _build/wasm-gc/release/build/main/main.wasm, ~280KB)
moon build --target wasm-gc --release

# MoonBit format
moon fmt

# Build TypeScript library (output: dist/)
tsc

# Build everything (Wasm + TypeScript)
npm run build

# Run all tests (MoonBit unit + Node.js integration)
npm test

# Run MoonBit unit tests only (xml, ooxml, renderer, svg_parser, serializer)
npm run test:moon

# Run Node.js integration tests only
npm run test:node

# Serve for browser testing
python3 -m http.server 8765 --directory .
# → http://localhost:8765/web/index.html
```

## Architecture

**Separation of concerns:**
- **TypeScript library** (`lib/` → `dist/`): ZIP parsing/building, DEFLATE, Wasm lifecycle, CRC-32, EMF→SVG conversion, `PptxRenderer` class
- **MoonBit** (`src/`): OOXML parsing, SVG generation, SVG→SlideData parsing, OOXML serialization
- **Demo** (`web/`): Browser demo UI (imports from `dist/`)

**FFI boundary:**
- JS pre-decompresses all ZIP entries → stores in `Map<path, string>` and `Map<path, Uint8Array>`
- MoonBit calls `ffi_get_file(path)` to pull individual files on demand
- MoonBit exports (read-only): `initialize_pptx`, `get_slide_count`, `is_slide_hidden`, `get_slide_xml_raw`, `get_entry_list`, `render_slide_svg`, `update_slide_from_svg`, `get_slide_ooxml`, `get_modified_entries`
- MoonBit exports (editing): `render_shape_svg`, `update_shape_transform`, `update_shape_text`, `update_shape_fill`, `delete_shape`, `add_shape`, `add_shape_text`, `duplicate_shape`, `update_shape_gradient_fill`, `update_shape_stroke`, `add_paragraph`, `delete_paragraph`, `add_run`, `delete_run`, `update_text_run_style`, `update_text_run_font_size`, `update_text_run_color`, `update_text_run_font`, `update_paragraph_align`, `update_text_run_decoration`, `add_picture_shape`, `replace_picture_rid`
- MoonBit exports (history E6.1): `restore_slide_ooxml`
- MoonBit exports (inline text editing E6.2): `get_text_layout`, `hit_test_text`, `replace_text_range`
- MoonBit exports (z-order E6.3): `bring_to_front`, `send_to_back`, `bring_forward`, `send_backward`
- MoonBit exports (multi-transform E6.4): `update_shapes_transform`
- MoonBit exports (copy/paste E6.5): `get_shape_ooxml`, `add_shape_from_ooxml`
- MoonBit exports (table editing E6.6): `update_table_cell_text`, `add_table_row`, `delete_table_row`, `add_table_column`, `delete_table_column`
- Full export list: see `src/main/moon.pkg`

**Lazy slide parse (`g_slides` cache).** `initialize_pptx` fills `g_slides` with empty placeholders (`shapes: []`); the real parse + placeholder inheritance happens on the first `render_slide_svg(idx)`, which caches the resolved `SlideData` and sets `g_parsed[idx]`. Editing exports read `g_slides` directly, so any new editing export that reads the cache **must** route through `with_shape`/`with_run` (which call `ensure_slide_parsed`) or call `ensure_slide_parsed(slide_idx)` itself — otherwise it silently no-ops on a slide that was never rendered (this was the 0.5.10 bug). Don't add per-method `renderSlideSvg()` "ensure parsed" calls in the TS layer; the Wasm boundary is the single source of truth.

**Module dependency (no cycles):**
```
main → renderer   → xml, ooxml, ffi
     → svg_parser → xml, ooxml, ffi
     → serializer → xml, ooxml, ffi
     → ffi
xml (shared: int_to_str, parse_int, XML parser)
ooxml → xml (types, PPTX parser, parse_hex_color)
```

## Critical MoonBit constraints

**No integer string interpolation.** `"\{n}"` for integer `n` calls `fromCharCodeArray` internally, which requires `{ builtins: ['js-string'] }` browser support (Chrome 117+). The codebase uses `@xml.int_to_str(n)` (defined in `xml.mbt`, aliased locally as `fn int_to_str(n) -> String { @xml.int_to_str(n) }`) which only uses `concat` + string literals and works in all wasm-gc browsers (Chrome 111+).

**String API:** Use `s.get_char(i).unwrap()` (not deprecated `unsafe_char_at`). Avoid `s[i:j]` in non-error functions — it raises `CreatingViewError`.

**No external packages.** `bobzhang/zip` and `ruifeng/XMLParser` are incompatible with the current compiler (Feb 2026). Do not add external deps; implement needed parsers inline.

**pub(all) for cross-package construction.** Structs and enums in `ooxml` that need to be constructed from other packages (svg_parser, serializer, main) use `pub(all)` visibility. `pub struct` fields are read-only from other packages.

**Watch Int32 overflow in geometry math.** `Int` is 32-bit. `(x2 - x1) * adj / 100000` overflows when both operands are large in opposite signs (e.g. Google Slides writes connector adj1 = -39687500, paired with multi-million-EMU spans). Use `to_int64()` for the multiplication and divide back, or split as `span / 100 * adj / 1000`. See `bend_offset` in `renderer.mbt`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [t-ujiie-g/pptx-svg](https://github.com/t-ujiie-g/pptx-svg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
