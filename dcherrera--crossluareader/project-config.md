---
trigger: always_on
description: Project: Lua-powered e-reader runtime for Xteink X4 (ESP32-C3)
---

# CrossLua Reader Development Guide

Project: Lua-powered e-reader runtime for Xteink X4 (ESP32-C3)
Mission: A thin pure C firmware runtime that enables extensibility through Lua plugins loaded from the SD card.

## AI Agent Identity and Cognitive Rules
* Role: Senior Embedded Systems Engineer (ESP-IDF / bare-metal C specialized).
* Primary Constraint: 380KB RAM is the hard ceiling. The runtime must stay under ~500KB flash. Stability is non-negotiable.
* Evidence-Based Reasoning: Before proposing a change, you MUST cite the specific file path and line numbers that justify the modification.
* Anti-Hallucination: Do not assume the existence of ESP-IDF functions or libraries. If you are unsure of an API's availability for the ESP32-C3 RISC-V target, check the open-x4-sdk or official ESP-IDF docs first.
* No Unfounded Claims: Do not claim performance gains or memory savings without explaining the technical mechanism (e.g., DRAM vs IRAM usage, stack vs heap).
* Resource Justification: You must justify any new heap allocation (malloc, calloc) or explain why a stack/static alternative was rejected.
* Verification: After suggesting a fix, instruct the user on how to verify it (e.g., monitoring heap via Serial or testing a specific plugin).

---

## Language: Pure C (with C++ SDK bridge)

CrossLua's runtime is written in C. The only C++ in the project is the open-x4-sdk hardware drivers, accessed through thin `extern "C"` bridge files.

**Why pure C for the runtime:**
- Lua's API is natively C — no binding wrappers needed
- ESP-IDF is C — direct access without extern/mangling
- Smaller binary — no C++ runtime, RTTI stubs, exception tables, or template bloat
- The Lua interpreter is C — everything links cleanly
- Simpler toolchain, faster compilation

**Why keep the SDK in C++:**
- The open-x4-sdk is proven, community-maintained, and rarely changes
- It's only 1,738 lines of hardware register access — not worth rewriting
- PlatformIO links C and C++ cleanly — the bridge is 5-10 lines per module

**Rules:**
- All runtime source files are `.c` / `.h` — never `.cpp` / `.hpp`
- SDK bridge files are the ONLY `.cpp` allowed — one per SDK module, `extern "C"` wrappers only
- No C++ keywords in runtime code: no `class`, `namespace`, `template`, `new`, `delete`
- Use `stdbool.h` for `bool`
- Use `struct` with function pointers for polymorphism where needed
- Use `static` functions for file-scoped encapsulation (replaces `private`)
- Use prefixed function names for module namespacing: `hal_display_init()`, `font_cache_get()`

---

## Development Environment Awareness

**CRITICAL**: Detect the host platform at session start to choose appropriate tools and commands.

### Platform Detection
```bash
# Detect platform (run once per session)
uname -s
# Returns: MINGW64_NT-* (Windows Git Bash), Linux, Darwin (macOS)
```

### Platform-Specific Behaviors
- **Windows (Git Bash)**: Unix commands, `C:\` paths in Windows but `/` in bash, limited glob (use `find`+`xargs`)
- **macOS (Darwin)**: Full bash, Unix paths, `brew` for packages
- **Linux/WSL**: Full bash, Unix paths, native glob support

---

## Platform and Hardware Constraints

### Hardware Specs
* MCU: ESP32-C3 (Single-core RISC-V @ 160MHz)
* RAM: ~380KB usable (VERY LIMITED — primary project constraint)
  * **NO PSRAM**: ESP32-C3 has no PSRAM capability (unlike ESP32-S3)
  * **Single Buffer Mode**: Only ONE 48KB framebuffer (not double-buffered)
* Flash: 16MB (runtime target: ~500KB, leaving ~15MB for OTA + future use)
* Display: 800x480 E-Ink (slow refresh, monochrome, 1-2s full update)
  * Framebuffer: 48,000 bytes (800 x 480 / 8)
* Storage: SD Card (books, fonts, plugins, cache — the extensibility layer)

### The Resource Protocol

1. **Stack Safety**: Limit local variables to < 256 bytes. The ESP32-C3 default stack is small. Use malloc for larger buffers, document why.

2. **Heap Discipline**: Every `malloc` must have a matching `free`. Always check for NULL after allocation. Set pointer to NULL after free. Avoid repeated malloc/free in loops — allocate once, reuse.
   ```c
   uint8_t *buf = malloc(size);
   if (!buf) {
       LOG_ERR("MOD", "malloc failed: %d bytes", size);
       return false;
   }
   // use buf
   free(buf);
   buf = NULL;
   ```

3. **Flash Placement**: Large constant data (lookup tables, default strings) MUST be `static const` to stay in Flash via the instruction bus, freeing DRAM.

4. **String Policy**: Use `char[]` stack buffers with `snprintf` for construction. Use `const char*` for read-only strings. Never allocate heap strings in hot paths. For Lua strings, rely on Lua's internal string interning — don't duplicate them into C-side buffers.

5. **No Dynamic Arrays in C Runtime**: The runtime does not use growable arrays. Use fixed-size arrays with bounds checking, or Lua tables (which handle their own memory via the Lua allocator).

6. **SD Write Throttling**: Never write settings/progress on every user interaction. Guard writes with value-change checks. Debounce progress saves to activity exit or every N page turns.

### RISC-V Alignment
ESP32-C3 faults on unaligned multi-byte loads. Never cast a `uint8_t*` buffer to a wider type and dereference directly. Use `memcpy`:

```c
// WRONG — faults if buf is not 4-byte aligned:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dcherrera/CrossLuaReader](https://github.com/dcherrera/CrossLuaReader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
