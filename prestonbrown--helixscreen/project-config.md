---
trigger: always_on
description: **HelixScreen**: LVGL 9.5 touchscreen UI for Klipper 3D printers. XML engine in `lib/helix-xml/` (extracted from LVGL). Pattern: XML → Subjects → C++.
---

# CLAUDE.md

## Quick Start

**HelixScreen**: LVGL 9.5 touchscreen UI for Klipper 3D printers. XML engine in `lib/helix-xml/` (extracted from LVGL). Pattern: XML → Subjects → C++.

**Before compiling:** Check for existing build processes (`pgrep -f 'make|c\+\+'`) — concurrent compilations thrash the machine.

```bash
make -j                              # Build ONLY the program binary (NOT tests)
./build/bin/helix-screen --test -vv  # Mock printer + DEBUG logs
# ALWAYS use verbosity: -v=INFO, -vv=DEBUG, -vvv=TRACE (default=WARN)

make test                            # Build tests only (does NOT run them)
make test-run                        # Build AND run tests in parallel
./build/bin/helix-tests "[tag]"      # Run specific test tags
make pi-test                         # Build on thelio + deploy + run

# Worktrees — MUST use for MAJOR work. Always in .worktrees/ (project root).
scripts/setup-worktree.sh feature/my-branch  # Symlinks deps, builds fast
```

**XML hot reload:** `HELIX_HOT_RELOAD=1 ./build/bin/helix-screen --test -vv` — edit XML, save, switch panels to see changes live.

**Screenshots:** Press 'S' in UI, or `./scripts/screenshot.sh helix-screen output-name [panel]`

---

## Docs (load when needed)

Full index: **`docs/devel/CLAUDE.md`** (auto-loaded when working in docs/devel/)

Most commonly needed:

| Doc | When |
|-----|------|
| `docs/devel/UI_CONTRIBUTOR_GUIDE.md` | UI/layout work: breakpoints, tokens, colors, widgets, layout overrides |
| `docs/devel/LVGL9_XML_GUIDE.md` | XML layouts, widgets, bindings, observer cleanup |
| `docs/devel/MODAL_SYSTEM.md` | Modal architecture: ui_dialog, modal_button_row, Modal pattern |
| `docs/devel/FILAMENT_MANAGEMENT.md` | AMS, AFC, Happy Hare, ACE, AD5X IFS, CFS, Tool Changer |
| `docs/devel/ENVIRONMENT_VARIABLES.md` | Runtime env vars, mock config |
| `docs/devel/LOGGING.md` | spdlog levels: info vs debug vs trace |
| `docs/devel/BUILD_SYSTEM.md` | Makefile, cross-compilation |

---

## Code Standards

| Rule | ❌ WRONG | ✅ CORRECT |
|------|----------|-----------|
| **spdlog only** | `printf()`, `cout`, `LV_LOG_*` | `spdlog::info("temp: {}", t)` |
| **SPDX headers** | 20-line GPL boilerplate | `// SPDX-License-Identifier: GPL-3.0-or-later` |
| **RAII widgets** | `lv_malloc()` / `lv_free()` | `lvgl_make_unique<T>()` + `release()` |
| **Class-based** | `ui_panel_*_init()` functions | Classes: `MotionPanel`, `WiFiManager` |
| **Observer factory** | Static callback + `lv_observer_get_user_data()` | `observe_int_sync<Panel>()` from `observer_factory.h` |
| **Icon sync** | Add icon, forget fonts | codepoints.h + `make regen-fonts` + rebuild |
| **Formatting** | Manual formatting | Let pre-commit hook (clang-format) fix |
| **No auto-mock** | `if(!start()) return Mock()` | Check `RuntimeConfig::should_mock_*()` |
| **JSON include** | `#include <nlohmann/json.hpp>` | `#include "hv/json.hpp"` (libhv's bundled version) |
| **Build system** | `cmake`, `ninja` | `make -j` (pure Makefile) |
| **Bug commits** | `fix: thing` (no reference) | `fix(scope): thing (prestonbrown/helixscreen#123)` |

**ALWAYS:** Search the SAME FILE you're editing for similar patterns before implementing.

---

## CRITICAL RULES - Declarative UI

**DATA in C++, APPEARANCE in XML, Subjects connect them.**

| # | Rule | ❌ NEVER | ✅ ALWAYS |
|---|------|----------|----------|
| 1 | **NO lv_obj_add_event_cb()** | `lv_obj_add_event_cb(btn, cb)` | XML `<event_cb trigger="clicked" callback="name"/>` + `lv_xml_register_event_cb()` |
| 2 | **NO imperative visibility** | `lv_obj_add_flag(obj, HIDDEN)` | XML `<bind_flag_if_eq subject="state" flag="hidden" ref_value="0"/>` |
| 3 | **NO lv_label_set_text** | `lv_label_set_text(lbl, val)` | Subject binding: `<text_body bind_text="my_subject"/>` |
| 4 | **NO C++ styling** | `lv_obj_set_style_bg_color()` | XML: `style_bg_color="#card_bg"` |
| 5 | **NO manual LVGL cleanup** | `lv_display_delete()`, `lv_group_delete()` | Just `lv_deinit()` - handles everything |
| 6 | **bind_style priority** | `style_bg_color` + `bind_style` | Inline attrs override - use TWO bind_styles |
| 7 | **NO ad-hoc callback guards** | `shared_ptr<bool> alive_`, `callback_guard_`, `alive_guard_` | `lifetime_.token()` + `tok.defer(...)` via `AsyncLifetimeGuard` |
| 8 | **NO lifetime_.defer from BG** | `lifetime_.defer([this](){...})` in API callbacks | `tok.defer([this](){...})` — token holds its own shared_ptr (#707) |

**Exceptions:** DELETE cleanup, widget pool recycling, chart data, animations

---

## Design Tokens (MANDATORY)

| Category | ❌ WRONG | ✅ CORRECT |
|----------|----------|-----------|
| **Colors** | `lv_color_hex(0xE0E0E0)` | `ui_theme_get_color("card_bg")` |
| **Spacing** | `style_pad_all="12"` | `style_pad_all="#space_md"` |
| **Typography** | `<lv_label style_text_font="...">` | `<text_heading>`, `<text_body>`, `<text_small>` |

Note: `ui_theme_get_color()` for tokens, `ui_theme_parse_color()` for hex strings only (NOT tokens).

---

## Threading & Lifecycle

WebSocket/libhv callbacks = background thread. **NEVER** call `lv_subject_set_*()` directly.
Use `ui_queue_update()` from `ui_update_queue.h`. Pattern: `printer_state.cpp` `set_*_internal()`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prestonbrown/helixscreen](https://github.com/prestonbrown/helixscreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
