---
trigger: always_on
description: Adi is a GUI library in Ada 2022 using SDL3 for windowing/rendering and SDL3_ttf for text. It provides a widget-based UI framework with CSS-like styling, declarative XML UI generation, and runtime CSS live-reload.
---

# CLAUDE.md

## Project Overview

Adi is a GUI library in Ada 2022 using SDL3 for windowing/rendering and SDL3_ttf for text. It provides a widget-based UI framework with CSS-like styling, declarative XML UI generation, and runtime CSS live-reload.

## Read the Docs First

Before making changes, read the relevant documentation. Do not guess at APIs or conventions — the docs are authoritative.

| Topic | File |
|-------|------|
| First application, from an empty directory | `docs/getting_started.md` |
| Architecture & core components | `docs/architecture.md` |
| Handle ownership model (stores, handles, borrow, lifecycle) | `docs/handle_ownership.md` |
| Coding conventions | `docs/coding_conventions.md` |
| CSS styling (selectors, properties, runtime API, code generation) | `docs/css_styling.md` |
| XML UI system (declarative widgets, code generation, components) | `docs/xml_ui_system.md` |
| Internal style-storage optimization (interning/prepared rules/global memo) | `docs/style_storage_optimization.md` |
| Item-based flex layout inside a widget (`Layout_Item`) | `docs/layout_item_system.md` |
| Layout minimums (demanded vs. content vs. preferred size) | `docs/layout_minimums.md` |
| Hand-crafted SDL3 binding modules | `docs/sdl_bindings.md` |
| Build system (Alire, gprbuild, configure) | `docs/build.md` |
| gprbuild without Alire | `docs/gprbuild_without_alire.md` |
| MCP runtime introspection | `docs/mcp.md` |
| HTML view widget | `docs/html_view_spec.md` |
| Adding a new CSS property | `docs/adding_css_property.md` |
| Adding a new example | `docs/adding_example.md` |
| Adding a new test | `docs/adding_test.md` |
| Antialiased rendering (AA fringe, ring patterns) | `docs/rendering_aa.md` |
| OS integration (dialogs, paths, clipboard) | `docs/os_integration.md` |
| Static asset bundling | `docs/static_assets.md` |
| Signals and deferred dispatch | `docs/signals.md` |
| Internationalization (i18n, translations, .po files, plural forms) | `docs/i18n.md` |
| Program settings (Setting_Value, Settings_Store, JSON backend) | `docs/settings.md` |
| WebAssembly port (build, toolchain, JSPI, example status) | `wasm/README.md`, `wasm/PORT_REPORT.md`, `wasm/FINDINGS.md` |
| Finalization ordering issue (Window vs. widget tagged-type packages) and structural fix options | `docs/finalization_ordering.md` |
| Ada 2022 Reference Manual | `rm-22-txt/RM-TOC.TXT` (chapters: `rm-22-txt/RM-*.TXT`) — local-only, gitignored; if absent, fetch the plain-text RM into `rm-22-txt/` |

## Build Commands

> ⚠️ Build safety: never run more than one `gprbuild` command at the same time in this repo. Concurrent `gprbuild` processes can race on shared artifacts and produce truncated/corrupted archives.

```bash
# Build the library only (tests and examples are NOT built here)
alr build -- -j0

# Build + run the whole suite: Ada tests, Python tests, example widget-tree
# goldens. Also the alr test action. ADI_SKIP_TREE_GOLDENS=1 drops the
# goldens, which build and run every example and so cost about a minute.
alr exec -- tools/run_tests.sh

# Build a specific test
alr exec -- gprbuild -j0 -P tests/tests.gpr -XTEST_KIND=css_parser_test

# Build examples (regenerates their generated sources first)
tools/build_examples.sh stack_example

# Examples link libAdi.a statically: rebuild one after any src/ change,
# or it still runs the library it was built against.

# Build a specific example directly (generated sources must be current)
alr exec -- gprbuild -j0 -P examples/examples.gpr -XEXAMPLE_KIND=stack_example

# Run Ada tests (built to tests/bin/); one binary per Test_Kind in tests/tests.gpr,
# of which these are a sample
./tests/bin/styles
./tests/bin/layout_test
./tests/bin/css_parser_test
./tests/bin/css_source_test
./tests/bin/side_longhand_test
./tests/bin/text_buffer_test
./tests/bin/text_layout_test
./tests/bin/html_view_test
./tests/bin/disabled_test
./tests/bin/image_widget_test
./tests/bin/mcp_test
./tests/bin/bundle_test
./tests/bin/settings_test
./tests/bin/close_request_test
./tests/bin/text_editor_test
./tests/bin/dialog_test
./tests/bin/font_test
./tests/bin/scroll_primitives_test
./tests/bin/handle_store_test
./tests/bin/label_wrap_test
./tests/bin/clock_test
./tests/bin/texture_cache_test
./tests/bin/render_textures_test

# Check every example's widget tree against tests/goldens/trees/
# (rebuilds the examples; --update accepts what the apps report now)
alr exec -- tools/widget_trees.py
alr exec -- tools/widget_trees.py demo_flex

# Run Python tests (no build step needed)
python3 tools/test_css_to_ada.py
python3 tools/test_xml_to_ada.py
python3 tools/test_adi_mcp.py
python3 tools/test_binary_to_ada.py
python3 tools/test_po_to_ada.py
```

For direct gprbuild (no Alire), see `docs/gprbuild_without_alire.md` and `docs/build.md`.

## Code Generation Pipelines

### CSS → Ada (`tools/css_to_ada.py`)

```bash
python3 tools/css_to_ada.py input.css output.ads --package-name=My_Styles
```

Writes a companion `output.adb` beside the spec, holding the stylesheet's
`Register_Selectors` procedure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ovenpasta/adi2](https://github.com/ovenpasta/adi2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
