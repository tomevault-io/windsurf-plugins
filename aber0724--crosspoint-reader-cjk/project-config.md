---
trigger: always_on
description: This file is for autonomous coding agents working in `crosspoint-reader-cjk`.
---

# AGENTS.md

This file is for autonomous coding agents working in `crosspoint-reader-cjk`.
It captures practical commands, style conventions, and repo guardrails.

## 1) Project Snapshot

- Target: Xteink X4 firmware (ESP32-C3), PlatformIO + Arduino.
- Main language: C++ (plus Python utility/generator scripts).
- Entry point: `src/main.cpp`.
- CI baseline: format, static analysis, and build.
- Build-time generators are wired into PlatformIO `extra_scripts`:
  - `scripts/build_html.py`
  - `scripts/gen_i18n.py`

## 2) Environment Setup

- Required tools: `python3`, `pio`, `clang-format` 21+.
- CI currently uses Python 3.14.
- Clone with submodules:
  - `git clone --recursive <repo-url>`
  - `git submodule update --init --recursive`

## 3) Build, Lint, Test Commands

### Primary Local Commands

- Format tracked C/C++ files: `./bin/clang-format-fix`
- Format only modified tracked files: `./bin/clang-format-fix -g`
- Static analysis (cppcheck): `pio check --fail-on-defect low --fail-on-defect medium --fail-on-defect high`
- Build default firmware: `pio run`
- Build explicit default env: `pio run -e default`
- Flash to device: `pio run --target upload`
- Serial monitor: `pio device monitor`

### Release Build Commands

- Simplified Chinese release env: `pio run -e gh_release`
- Traditional Chinese release env: `pio run -e gh_release_tc`

### Test Commands (Important)

- There is no regular `pio test` suite enforced in CI at this time.
- The active test path is hyphenation evaluation via `test/run_hyphenation_eval.sh`.
- Run full hyphenation summary (all supported languages):
  - `./test/run_hyphenation_eval.sh`
- Run a single test target (single language):
  - `./test/run_hyphenation_eval.sh english`
  - `./test/run_hyphenation_eval.sh french`
  - `./test/run_hyphenation_eval.sh german`
  - `./test/run_hyphenation_eval.sh russian`
  - `./test/run_hyphenation_eval.sh spanish`
  - `./test/run_hyphenation_eval.sh italian`

### Recommended Pre-PR Verification

- `./bin/clang-format-fix`
- `pio check --fail-on-defect low --fail-on-defect medium --fail-on-defect high`
- `pio run`

## 4) Code Style And Conventions

### Formatting

- `.clang-format` is authoritative; do not hand-format.
- Key settings: 2-space indent, 120 column limit, attached braces, left pointer alignment (`Type* ptr`), max 1 empty line.
- Include sorting/regrouping is enabled; let clang-format reorder includes.

### Include Ordering

- Preferred include shape in `.cpp` files:
  1. Matching local header first (`#include "Foo.h"`)
  2. External/platform headers (`<...>`)
  3. C/C++ stdlib headers (`<cstring>`, `<string>`, etc.)
  4. Other project headers (`"..."`)
- Avoid `using namespace ...` in headers and sources.
- Include what you use; keep headers minimal.

### Types, Ownership, And Strings

- Prefer fixed-width integer types for persisted/protocol/config fields (`uint8_t`, `uint16_t`, ...).
- Use `size_t` for sizes/indices.
- Prefer `const` correctness and pass-by-reference to avoid copies.
- For optional/owned heap objects, prefer `std::unique_ptr`.
- Use `std::string` for internal C++ logic.
- Use Arduino `String` at Arduino/WebServer API boundaries.

### Naming

- Classes/types: `PascalCase` (e.g., `CrossPointWebServer`).
- Functions/methods/variables: `camelCase` (e.g., `handleFileListData`).
- Enums/constants/macros: usually `UPPER_SNAKE_CASE`.
- `constexpr` constants in tests/utilities may use `kPrefix` (e.g., `kSupportedLanguages`).
- File names are commonly `PascalCase.h/.cpp` for class-oriented code.

### Error Handling And Logging

- Prefer early-return guard clauses.
- Check IO/network/storage return values immediately and fail explicitly.
- Use logging macros consistently:
  - `LOG_DBG(tag, ...)`
  - `LOG_INF(tag, ...)`
  - `LOG_ERR(tag, ...)`
- Keep log tags short and consistent (`"MAIN"`, `"WEB"`, `"CPS"`, etc.).
- For fixed char buffers, after `strncpy` always force null termination.

### Embedded Runtime Constraints

- Avoid unnecessary allocations in hot paths.
- Keep loops watchdog-safe (`yield()` / watchdog reset where appropriate).
- Respect power/sleep semantics in the main activity loop.
- Be cautious with SD-card IO and RAM pressure.

### Dark Mode Refresh Policy

- In dark mode, do not introduce new reader/UI paths that use `HALF_REFRESH` or ordinary `FAST_REFRESH` for visible updates.
- Prefer `DARK_REDRIVE` for dark-mode updates so the controller re-drives all pixels and the dark background does not fade/whiten.
- If adding a new refresh decision, activity transition, cadence/full-refresh replacement, partial/window update, or low-memory fallback, explicitly handle `darkMode` and route it to `RefreshMode::DarkRedrive`, `displayBufferDarkRedrive()`, or `displayWindowDarkRedrive()` as appropriate.
- Only use `HALF_REFRESH` in dark mode with a documented hardware reason and after device testing confirms it does not make the background turn white.

### Global Access Patterns

- This codebase intentionally uses singleton-style globals/macros such as `SETTINGS`, `APP_STATE`, and `I18N`.
- Follow existing patterns rather than introducing a new DI architecture in isolated edits.

## 5) Generated And Protected Areas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aBER0724/crosspoint-reader-cjk](https://github.com/aBER0724/crosspoint-reader-cjk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
