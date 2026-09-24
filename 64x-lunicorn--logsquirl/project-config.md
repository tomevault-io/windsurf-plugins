---
trigger: always_on
description: LogSquirl is a cross-platform log viewer built with C++23 and Qt6.
---

# LogSquirl — Copilot Instructions

## Project Overview

LogSquirl is a cross-platform log viewer built with C++23 and Qt6.
It is a GPL-3.0-or-later licensed fork of [klogg](https://github.com/variar/klogg), which itself
is a fork of [glogg](https://github.com/nickbnf/glogg). The build system is CMake (minimum 3.16).
Dependencies are managed via [CPM](https://github.com/cpm-cmake/CPM.cmake).

- **Repository**: <https://github.com/64x-lunicorn/LogSquirl>
- **License**: GPL-3.0-or-later (see `COPYING`)
- **Bundle identifier**: `io.github.logsquirl`

## Language

**All code, comments, documentation, commit messages, issues, and pull requests must be written
in English.** No exceptions.

## Documentation

**Every change must be documented.** This includes:

- New or modified public functions/methods **must** have a short comment above the declaration
  explaining purpose, parameters, and return value.
- Non-trivial logic **must** have inline comments explaining *why*, not *what*.
- New files **must** include the GPL-3.0 license header (see "File Headers" below).
- User-facing changes **must** be noted in `CHANGELOG.md`.
- Build or dependency changes **must** be reflected in `BUILD.md`.
- Architecture or structural changes **must** be described in the commit message body.

## C++ Code Style

The project uses `.clang-format` and `.clang-tidy` at the repository root. Always format code
with clang-format before committing.

### Naming Conventions (enforced by `.clang-tidy`)

| Element              | Convention        | Example                              |
|----------------------|-------------------|--------------------------------------|
| Namespace            | `lower_case`      | `logsquirl::`                        |
| Class / Struct       | `CamelCase`       | `CrawlerWidget`, `LogData`           |
| Function / Method    | `camelBack`       | `getTopLine()`, `displayQuickFind()` |
| Variable / Parameter | `camelBack`       | `lineNumber`, `fileSize`             |
| Private member       | `camelBack_`      | `ignoreCase_`, `autoRefresh_`        |
| Global constant      | `CamelCase`       | `MaxRecentFiles`, `ErrorPalette`     |
| Macro / Define       | `UPPER_SNAKE_CASE`| `LOGSQUIRL_USE_LTO`                  |
| Enum value           | `CamelCase`       | `SearchRegexpType`                   |

### Formatting Highlights (from `.clang-format`)

- **Standard**: C++23
- **Column limit**: 100
- **Indent**: 4 spaces (no tabs)
- **Braces**: Custom — opening brace on next line for functions; `else` on new line
- **Pointer/Reference alignment**: Left (`int* ptr`, `const QString& str`)
- **Spaces in parentheses**: Yes — `if ( condition )`, `foo( arg )`
- **Spaces in square brackets**: Yes — `arr[ i ]`
- **Constructor initializers**: Break before comma
- **Template declarations**: Always break after template

### Header Guards

- Prefer `#pragma once` for new files.
- Existing `#ifndef` guards follow the pattern `LOGSQUIRL_FILENAME_H`.

### Include Order

1. Corresponding header for `.cpp` files
2. Project headers (`#include "log.h"`)
3. Qt headers (`#include <QApplication>`)
4. Standard library headers (`#include <algorithm>`)
5. Third-party headers

### Modern C++ Usage

- Use `auto` where the type is obvious from context.
- Use `const` and `constexpr` liberally — mark everything `const` that can be.
- Use range-based for loops: `for ( const auto& item : collection )`.
- Use `if constexpr` for compile-time branching.
- Use strong typedefs (`type_safe::strong_typedef`) for domain types like line numbers and offsets.
- Prefer smart pointers (`std::unique_ptr`, `std::shared_ptr`) over raw owning pointers.

## Qt Patterns

- **Signal/Slot connections**: Use new-style `connect()` with function pointers:
  ```cpp
  connect( sender, &Sender::signal, receiver, &Receiver::slot );
  ```
- **Cross-thread connections**: Explicitly specify `Qt::QueuedConnection`.
- **Meta-object macros**: Use `Q_OBJECT`, `Q_SIGNALS`, `Q_SLOTS` (not `signals:` / `slots:`).
- **Meta-type registration**: Register custom types used across threads with `qRegisterMetaType`.
- **Qt version**: The codebase targets Qt6 only. Do not add Qt5 compatibility code.
- **UI files**: Use Qt Designer `.ui` files for dialog layouts. Build system has `AUTOUIC ON`.
- **Resources**: Use `.qrc` files with `AUTORCC ON`.

## CMake Conventions

### Variable and Option Naming

- Project options: `LOGSQUIRL_<FEATURE>` (e.g., `LOGSQUIRL_BUILD_TESTS`, `LOGSQUIRL_USE_LTO`)
- Internal variables: `UPPER_SNAKE_CASE`

### Target Naming

- Libraries: `logsquirl_<module>` (e.g., `logsquirl_ui`, `logsquirl_logdata`, `logsquirl_utils`)
- Executables: `logsquirl`, `logsquirl_portable`, `logsquirl_grep`
- Test targets: `logsquirl_tests`, `logsquirl_compression_tests`, `logsquirl_plugin_catalog_tests`,
  `logsquirl_openlogfile_tests`, `logsquirl_textviewscrolling_tests`, `logsquirl_versioncheck_tests`, `logsquirl_itests`
- CTest script checks: `logdata_public_headers_no_tbb`, `plugins_no_qt_widgets`,
  `openlogfile_no_qt_widgets`, `textviewscrolling_no_qt_widgets`, `ui_settings_store_allowlist`,
  `logsquirl_grep_cli`

### Module Structure

Each module in `src/` follows this layout:

```
src/<module>/
├── CMakeLists.txt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [64x-lunicorn/LogSquirl](https://github.com/64x-lunicorn/LogSquirl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
