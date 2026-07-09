---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the `std_module` repository.
---

# CLAUDE.md - AI Assistant Guide for std_module

This document provides comprehensive guidance for AI assistants working with the `std_module` repository.

## Project Overview

**std_module** is a C++20 module wrapper library for the C++ standard library. It enables developers to use modern `import` statements instead of traditional `#include` directives.

- **Language:** C++20 with modules
- **Version:** 0.1.0
- **Build System:** CMake 3.28+ with Ninja (required)
- **Status:** Production-ready - 72 standard library modules implemented with comprehensive tests

### Core Philosophy

The project prioritizes **flexibility over opinion**:
- No forced dependencies between modules
- Opt-in architecture - build only what you need
- Multiple integration methods (subdirectory, installed, manual)
- Standard library semantics preserved exactly (no modifications)

## Repository Structure

```
/home/user/std_module/
├── CMakeLists.txt              # Root build config
├── README.md                   # User-facing documentation (lean, pattern-focused)
├── CLAUDE.md                   # This file - comprehensive AI guide
├── src/                        # Module implementations (72 .cppm files)
│   ├── CMakeLists.txt         # Module build targets (uses macros)
│   ├── format.cppm            # Example: <format> wrapper
│   ├── vector.cppm            # Example: <vector> wrapper
│   ⋮
├── test/                       # Test suite (72 test files)
│   ├── CMakeLists.txt         # Test build config (uses macros)
│   ├── README.md              # Test suite documentation
│   ├── test_format.cpp        # Example: comprehensive format tests
│   ⋮
├── cmake/                      # CMake infrastructure
│   ├── StdModuleMacros.cmake  # Helper macros for adding modules
│   └── std_module-config.cmake.in
└── scripts/                    # Automation scripts
    ├── README.md
    └── symbol_coverage.py     # Symbol coverage analysis tool
```

### Implemented Modules (72 Total)

**Note:** README.md contains the canonical, exhaustive list of all 72 implemented modules in the "Available Modules" table. That table is the **single source of truth** for module availability, status, and special notes.

**Categories overview:**
- **Algorithms & Iterators:** algorithm, functional, iterator, ranges
- **Containers:** any, bitset, deque, forward_list, initializer_list, list, map, optional, queue, span, variant, vector
- **I/O:** fstream, ios, iosfwd, iostream, istream, iomanip (⚠️), syncstream
- **Text & Formatting:** charconv, format, locale, string_view
- **Concurrency:** barrier, condition_variable, coroutine, future (⚠️), latch, semaphore
- **Utilities:** bit, compare, concepts, exception, execution, filesystem, limits, memory_resource, new, numbers, numeric, random, source_location, system_error, typeindex

**Special cases:**
- `<new>` → `std_module.new_` (underscore to avoid keyword conflict)
- `<iomanip>` → ⚠️ Manipulators unusable due to ADL limitations
- `<future>` → ⚠️ `packaged_task` unusable, other components work

### README.md Documentation Philosophy

**IMPORTANT:** The README.md follows a **lean, pattern-focused approach**. This is a deliberate design choice to keep user-facing documentation concise and maintainable.

**Key principles:**

1. **Describe patterns, not exhaustive lists**
   - CMake options: Show the pattern `STD_MODULE_BUILD_<NAME>`, list the 3 special options, note defaults
   - Library targets: Show the pattern `std_module::<name>`, give 3-4 examples
   - **Don't** create tables listing all 72 modules in multiple places

2. **Single source of truth**
   - The "Available Modules" table is the **ONLY** place that exhaustively lists all 72 modules
   - Everything else refers to this table
   - When adding a module, update **only** this table in README.md

3. **Assume user knowledge**
   - **Don't** explain what `<vector>` does or what features `<algorithm>` provides
   - Users of this library already know the standard library
   - Focus on: module name, import statement, build options, known limitations

4. **Use vertical ellipses in file trees**
   - Example: `├── format.cppm`, `├── vector.cppm`, `⋮`
   - Saves space while conveying "many files here"

5. **Be concise everywhere except "Available Modules"**
   - Short paragraphs
   - Minimal examples
   - Only essential information

**What this means when updating README.md:**

- ✅ Add new module to "Available Modules" table with status and notes
- ✅ Keep examples to 3-4 items max (don't list all 72 modules)
- ✅ Use patterns instead of exhaustive enumerations
- ❌ Don't add the module to CMake Options table (removed - uses pattern now)
- ❌ Don't add the module to Library Targets table (removed - uses pattern now)
- ❌ Don't describe what the standard library component does
- ❌ Don't create new exhaustive lists

**Rationale:** With 72 modules (and growing toward 90+), exhaustive tables become:
- Hard to maintain (update in N places for each new module)
- Noisy for users (too much scrolling)
- Redundant (same information repeated multiple ways)

**This file (CLAUDE.md)** remains comprehensive and detailed - it's the AI assistant guide. README.md is for human users who want to get started quickly.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whatDiracIsCooking/std_module](https://github.com/whatDiracIsCooking/std_module) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
