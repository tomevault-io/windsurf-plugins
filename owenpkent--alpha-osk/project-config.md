---
trigger: always_on
description: Alpha-OSK is an AI-assisted, mouse-driven on-screen keyboard for Windows and Linux (macOS in progress). Users click QML keys to type into whatever app currently holds OS focus; a hybrid n-gram + PPM + fuzzy engine predicts words locally, with no LLM, no GPU, and nothing leaving the machine. It is an accessibility tool the owner depends on daily. This file is both the AI-onboarding doc and the human codebase map; the detailed reference sections below are authoritative project knowledge, not backg
---

# CLAUDE.md: Alpha-OSK AI Onboarding

Alpha-OSK is an AI-assisted, mouse-driven on-screen keyboard for Windows and Linux (macOS in progress). Users click QML keys to type into whatever app currently holds OS focus; a hybrid n-gram + PPM + fuzzy engine predicts words locally, with no LLM, no GPU, and nothing leaving the machine. It is an accessibility tool the owner depends on daily. This file is both the AI-onboarding doc and the human codebase map; the detailed reference sections below are authoritative project knowledge, not background.

## Key rules (non-obvious, cross-cutting)

- The keyboard must NEVER steal OS focus: `WS_EX_NOACTIVATE` on Windows (`keyboard_app.py::_apply_window_flags`), `WindowDoesNotAcceptFocus` elsewhere. Because our window cannot hold focus, route in-app text entry (prediction-edit popup, snippets editor, any future input slot) through `setEditMode(true)` plus the `editKeyTyped` / `editSpecialPressed` signals, never Qt focus. Set edit mode on open and clear it on close.
- Sticky-modifier auto-release logic is duplicated in `_press_char` and `pressSpecialKey` (state flip + `release_modifier()` + change-signal emit, plus `_update_layer()` for Shift). Keep both blocks in sync; new keystroke paths (autocorrect retype, pill insert, macros) must mirror it. `pressSpecialKey` deliberately keeps Shift/Ctrl held on `_NAV_KEYS` (arrows/home/end/pageup/pagedown).
- Linux `LinuxKeySynthesizer.hold_modifier()` MUST skip `win`/`super`: holding Super triggers a WM pointer grab that swallows every click, including clicks on the OSK itself. Do not "fix" it to hold Super. Windows still holds `VK_LWIN`.
- Pill-facing casing comes only from `KeyboardBridge._display_cased`, which mirrors every uppercase position of the typed prefix onto the pill, unconditionally (including fuzzy/autocorrect candidates). Auto-capitalisation is ONLY the "I" family in `ngram_predictor._always_capitalize`; do NOT reintroduce the removed three-tier proper-noun auto-cap as a default. Every pill emit site must route through `_display_cased`.
- Prediction insertion is suffix-only (type just the unseen tail), falling back to `replace_text()` on a prefix/casing mismatch. Compatibility Mode (`_in_compat_mode`, matched on IDE/RDP exe basenames in `_COMPAT_PROCESS_NAMES`, never window class) rewires this to BackSpace+retype. `_context_buffer` / `_current_word` must always mirror the on-screen text; backspace must trim and rehydrate a mid-word tail.
- Import paths are security-critical: `PackManager.import_pack`, `data_export.import_user_data`, and `inspect_export` sanitise names, cap sizes, and use allow-list (not deny-list) extraction against zip-slip. Do NOT loosen without re-reading the regression tests (`tests/test_vocabulary_pack.py::TestImportPackSecurity`, and the slip/absolute-path/oversize/future-schema/telemetry cases in `tests/test_data_export.py`).
- Privacy/password mode must suppress learning AND `activeContextChanged` so no password characters or password-field context leak into predictions, telemetry, or the live visualization. Detection is Windows UIA COM + Win32 fallback (`src/platform/password_detect.py`) and Linux AT-SPI2.
- Telemetry is OFF by default and `DEFAULT_ENDPOINT` in `src/telemetry.py` ships empty (silent no-op). `TelemetryClient` is the source of truth for the consent flag; do NOT mirror it into `appSettings`. The Data Backup archive deliberately excludes `telemetry.json`.
- Adding a setting requires the full 8-step wiring (see "Settings Panel Structure"): `Settings{}` savedFoo + root prop in `Main.qml`, prop + `SettingsToggle` in the correct sub-view of `UnifiedSettingsPanel.qml`, pass-through, `onSettingChanged`, optional `@Slot` on `keyboard_bridge.py`, and load in `Component.onCompleted`.
- Releases: `src/__version__.py` is the single source of version truth; publish to the separate `okstudio1/alpha-osk-releases` repo with an explicit `--repo` (the updater API URL is hard-pinned there); the installer asset name must be exactly `Alpha-OSK-Setup-{version}.exe`.
- Load-bearing invariants: merge-strategy default MUST stay `"rank"`; `NgramPredictor._user_total == sum(user_vocab.values())`; window height is content-bound (never persist or assign it); every analytics metric needs both a session and an `_alltime_*` form; Windows subprocess calls that suppress output need `CREATE_NO_WINDOW`.

## Stack & layout

- Python 3.10+ backend (CI runs 3.11, mypy targets 3.10), PySide6 (Qt6) + QML UI. No LLM/GPU. Key synthesis: ctypes SendInput scancode mode (Windows), `xdotool`/`ydotool` subprocess (Linux, NOT bundled), Quartz CGEvent (macOS, WIP).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owenpkent/alpha-osk](https://github.com/owenpkent/alpha-osk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
