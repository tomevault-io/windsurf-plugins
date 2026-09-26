---
trigger: always_on
description: Before analyzing the code, check whether `compile_commands.json` exists in this
---

# APPLaunch development guidance

## Before code analysis: ensure `compile_commands.json` exists

Before analyzing the code, check whether `compile_commands.json` exists in this
directory (`projects/APPLaunch/compile_commands.json`). If it does not, follow
the cross-compilation instructions in `../../README_ZH.md` (repo root): load
the config matching your current platform, then run one cross-compilation:

```bash
export CONFIG_DEFAULT_FILE=linux_x86_cross_cp0_config_defaults.mk
bbear -- scons -j22
```

`linux_x86_cross_cp0_config_defaults.mk` corresponds to the current platform
(Linux x86_64 host, CP0 cross toolchain). Pick the matching
`*_config_defaults.mk` for your platform from `../../README_ZH.md` — for
example `linux_x86_sdl2_config_defaults.mk` for the SDL2 simulator on Linux.

This command generates `compile_commands.json` in this directory. Then use the
source files that actually participate in the cross-compilation (as listed in
`compile_commands.json`) to confirm which code the project really uses, before
working on the coding task.

## Scoped-enum conversions

`cp0_lvgl` exports the public header `cp0_enum_cast.h`. Include it directly in
any C++ file that uses the conversion macro:

```cpp
#include "cp0_enum_cast.h"
```

Use the general `CP0_ENUM_CAST(target_type, enum_value)` for an explicit
conversion from an `enum class` value. Convenience macros are available for
common targets, including `CP0_ENUM_CAST_INT`, `CP0_ENUM_CAST_SIZE_T`,
`CP0_ENUM_CAST_UINT8`, `CP0_ENUM_CAST_UINT16`, `CP0_ENUM_CAST_UINT32`, and
`CP0_ENUM_CAST_UINT64`:

```cpp
enum class LayoutMetric : int { Width = 320 };

constexpr int width = CP0_ENUM_CAST_INT(LayoutMetric::Width);
constexpr auto count = CP0_ENUM_CAST_SIZE_T(LayoutMetric::Width);
constexpr auto raw = CP0_ENUM_CAST(uint32_t, LayoutMetric::Width);
```

This macro is the shared replacement for repeated enum-only `static_cast`
expressions and conversion helpers that only wrap an enum cast. Use the
type-specific convenience macro when it exists; use the general macro for any
other target type. Do not add a project-local helper solely to wrap one of
these macros; if an existing helper is part of a broader API, keep it and use
the macro in its implementation.

Do not use these macros for pointer casts, arbitrary integer conversions, or
untrusted values read from an external API. Validate and range-check an
external integer before converting it to an enum. Pass a side-effect-free enum
value or enumerator as the macro argument, and do not redefine a project-local
macro with the same name.

The `cp0_lvgl` component publishes `include/` through its `SConstruct`
dependency, so consumers should include `cp0_enum_cast.h` by name rather than
using a relative path into `ext_components`. After migrating a conversion,
keep the surrounding API and numeric behavior unchanged and run the APPLaunch
tests/build.

## Keyboard input systems

APPLaunch has two keyboard delivery systems. They share the same CP0 keyboard
backend and `key_item` queue; they are not two independent device readers. A
single physical, SDL, or injected key can be delivered through both paths:

1. The native LVGL path produces `LV_EVENT_KEY` for the focused object in the
   active input group.
2. The CP0 custom path sends `LV_EVENT_KEYBOARD` to the active screen with a
   complete `struct key_item` as the event parameter.

### Choose the event by capability

Use the native LVGL `LV_EVENT_KEY` path for ordinary UI operations: focus and
group navigation, list or menu movement, button activation, confirmation,
cancellation, and standard widget behavior. Bind the callback to the object
that belongs to the page input group and read the key with `lv_event_get_key()`:

```cpp
static void handle_key(lv_event_t *event)
{
    if (!event || lv_event_get_code(event) != LV_EVENT_KEY) return;
    const uint32_t key = lv_event_get_key(event);
    if (key == LV_KEY_ESC) close_page();
}
```

The native path receives the context-normalized key and CP0-to-LVGL mapping,
such as `LV_KEY_UP`, `LV_KEY_DOWN`, `LV_KEY_LEFT`, `LV_KEY_RIGHT`,
`LV_KEY_ENTER`, and `LV_KEY_ESC`. Make sure the keypad indev is assigned to the
page input group and that the intended object is focused. Prefer this path when
the operation needs no physical-key identity, text, modifiers, or explicit
press/release/repeat state.

Use the custom `LV_EVENT_KEYBOARD` path for text entry, Unicode input, terminal
input, application shortcuts, global shortcuts, physical-key-specific behavior,
modifier combinations, games, and any action that distinguishes pressed,
released, and repeated states. Register it on the current/root screen and read
the event parameter as a `const struct key_item *`. The CP0 backend registers
the shared event ID during input initialization; consumers must use the existing
`LV_EVENT_KEYBOARD` value rather than hard-coding or registering a second ID.

```cpp
static void handle_keyboard(lv_event_t *event)
{
    if (!event || lv_event_get_code(event) !=
            static_cast<lv_event_code_t>(LV_EVENT_KEYBOARD)) return;
    const auto *item = static_cast<const struct key_item *>(
        lv_event_get_param(event));
    if (!item || item->key_state != KBD_KEY_PRESSED) return;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CardputerZero/launcher](https://github.com/CardputerZero/launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
