---
trigger: always_on
description: Project: Open-source e-reader firmware for Xteink X4 (ESP32-C3)
---

# CrossPoint Reader Development Guide

Project: Open-source e-reader firmware for Xteink X4 (ESP32-C3)
Mission: Provide a lightweight, high-performance reading experience focused on EPUB rendering on constrained hardware.

## AI Agent Identity and Cognitive Rules

* Role: Senior Embedded Systems Engineer (ESP-IDF/Arduino-ESP32 specialized).
* Primary Constraint: 380KB RAM is the hard ceiling. Stability is non-negotiable.
* Evidence-Based Reasoning: Before proposing a change, you MUST cite the specific file path and line numbers that justify the modification.
* Anti-Hallucination: Do not assume the existence of libraries or ESP-IDF functions. If you are unsure of an API's availability for the ESP32-C3 RISC-V target, check the freeink-sdk source or the FreeInk SDK docs (https://freeink.org/llms.txt for an LLM-readable index) first.
* No Unfounded Claims: Do not claim performance gains or memory savings without explaining the technical mechanism (e.g., DRAM vs IRAM usage).
* Resource Justification: You must justify any new heap allocation (new, malloc, std::vector) or explain why a stack/static alternative was rejected.
* Verification: After suggesting a fix, instruct the user on how to verify it (e.g., monitoring heap via Serial or checking a specific cache file).

---

## Development Environment Awareness

**CRITICAL**: Detect the host platform at session start to choose appropriate tools and commands.

### Platform Detection

```bash
# Detect platform (run once per session)
uname -s
# Returns: MINGW64_NT-* (Windows Git Bash), Linux, Darwin (macOS)
```

**Detection Required**: Run `uname -s` at session start to determine platform

### Platform-Specific Behaviors

- **Windows (Git Bash)**: Unix commands, `C:\` paths in Windows but `/` in bash, limited glob (use `find`+`xargs`)
- **Linux/WSL**: Full bash, Unix paths, native glob support

**Cross-Platform Code Formatting**:

```bash
./bin/clang-format-fix -g
```

Never invoke or probe `clang-format` directly. The repository wrapper is the only sanctioned entry point.

---

## Platform and Hardware Constraints

### Hardware Specs

* MCUs: ESP32-C3 (single-core RISC-V @ 160MHz) and ESP32-S3 (`sticky`, dual-core Xtensa LX7)
* RAM: ~380KB usable on ESP32-C3 (VERY LIMITED - primary project constraint)
  * **NO PSRAM on C3**.
  * **Single Buffer Mode**: Only ONE 48KB framebuffer (not double-buffered)
* Flash: 16MB (Instruction storage and static data)
* Display: 800x480 E-Ink (Slow refresh, monochrome, 1-2s full update)
  * Framebuffer: 48,000 bytes (800 × 480 ÷ 8)
* Storage: SD Card (Used for books and aggressive caching)

### The Resource Protocol

1. Stack Safety: Limit local function variables to < 256 bytes. The ESP32-C3 default stack is small; use std::unique_ptr or static pools for larger buffers.
2. Heap Fragmentation: Avoid repeated new/delete in loops. Allocate buffers once during onEnter() and reuse them.
3. Flash Persistence: Large constant data (UI strings, lookup tables) MUST be marked static const to stay in Flash (Instruction Bus), freeing DRAM.
4. String Policy: Prohibit std::string and Arduino String in hot paths. Use std::string_view for read-only access and snprintf with fixed char[] buffers for construction.
5. UI Strings: All user-facing text must use the `tr()` macro (e.g., `tr(STR_LOADING)`) for i18n support. Never hardcode UI strings directly. For the avoidance of doubt, logging messages (LOG_DBG/LOG_ERR) can be hardcoded, but user-facing text must use `tr()`.
6. `constexpr` First: Compile-time constants and lookup tables must be `constexpr`, not just `static const`. This moves computation to compile time, enables dead-branch elimination, and guarantees flash placement. Use `static constexpr` for class-level constants.
7. `std::vector` Pre-allocation: Always call `.reserve(N)` before any `push_back()` loop. Each growth event allocates a new block (2×), copies all elements, then frees the old one — three heap operations that fragment DRAM. When the final size is unknown, estimate conservatively.
8. SD Persistence Throttling: Settings, state, credentials, and other `PersistableStore` JSON files live on SD under `/.crosspoint/` through `HalStorage`; SPIFFS is not mounted. Guard redundant writes and debounce progress saves to avoid serialization, SD I/O, and `storageMutex` cost.
9. `new` is not nothrow on ESP32: With `-fno-exceptions`, bare `new` that fails calls `abort()` — it does NOT return `nullptr`. Always use `new (std::nothrow)` and null-check the result, or use `makeUniqueNoThrow<T>()` from `lib/Memory/Memory.h`. Never write bare `new` for any fallible allocation.

---

## Project Architecture

### Build System: PlatformIO

**PlatformIO is BOTH a VS Code extension AND a CLI tool**:

1. **VS Code Extension** (Recommended):
   
   * Extension ID: `platformio.platformio-ide` (see `.vscode/extensions.json`)
   
   * Provides: Toolbar buttons, IntelliSense, integrated build/upload/monitor
   
   * Configuration: `.vscode/c_cpp_properties.json`, `.vscode/tasks.json`
   
   * Usage: Click Build (✓), Upload (→), or Monitor (🔌) buttons

2. **CLI Tool** (`pio` command):
   
   * **Installation**: Python package (typically `pip install platformio`)
   

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sfoulad/midad-by-foulad](https://github.com/sfoulad/midad-by-foulad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
