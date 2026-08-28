---
trigger: always_on
description: This is the canonical repo instruction file.
---

# CrossDiTo — Shared Agent Guide

This is the canonical repo instruction file.
`CLAUDE.md` should point here so Codex and Claude read the same guidance.

Project: Open-source e-reader firmware for the ESP32-S3-based Xteink X4 Pro.

## Core Rules

- Role: Senior Embedded Systems Engineer for ESP-IDF / Arduino-ESP32 work.
- Ship and validate only the Xteink X4 Pro. Preserve inherited shared-board code when removing it would add merge or regression risk, but do not publish other-device builds.
- Cite file paths and line numbers before proposing non-trivial changes.
- Do not assume ESP-IDF or SDK API availability. Verify in `freeink-sdk/` or the live code.
- Do not claim performance or memory wins without explaining the mechanism, such as reduced heap churn, flash vs DRAM placement, or smaller stack use.
- Justify new heap allocations or explain why stack/static storage is not suitable.
- Explain fixes in plain language where possible, ideally in terms a Node / React developer would follow.
- After proposing or making a fix, say how to verify it on hardware.

## Persistent Context

- Read `.claude/CONTEXT.md` at session start for durable repo-specific gotchas.
- Keep `.claude/CONTEXT.md` short. Add only reusable findings, not turn-by-turn history.

## Repo Skills

- Do not read every `.claude/skills/*/SKILL.md` at session start.
- Use this section as an index. Read a local skill only when the task clearly matches its folder name or purpose.
- Current local skills:
  - `control-flow-clarity`: simplify confusing logic without behavior changes.
  - `refactor-for-review`: small refactors intended to reduce review risk.
  - `hal-and-abstractions`: HAL boundaries and platform abstraction work.
  - `heap-discipline`: memory allocation, lifetime, and fragmentation-sensitive work.
  - `scope-discipline`: keep changes narrow and avoid unrelated cleanup.
  - `custom-fonts`: font generation, conversion, and SD/built-in font work.
- Treat these as task-specific playbooks layered on top of this guide. If a skill conflicts with this file, prefer `AGENTS.md` and note the conflict.

## Hardware Constraints

- Xteink X4 Pro uses an ESP32-S3R8: dual-core Xtensa at up to 240 MHz with 8 MB PSRAM. PSRAM is slower than internal DRAM and is not suitable for every DMA, ISR, or latency-sensitive buffer.
- Current displays use an 800x480 1-bit e-ink framebuffer: `800 * 480 / 8 = 48000` bytes. Use runtime renderer dimensions because orientation and future device profiles may differ.
- Use one framebuffer only, placed in PSRAM via `FREEINK_FB_PSRAM`.
- X4 Pro storage uses SDMMC and is exposed through SdFat. On real hardware, only one reader can hold a file open at a time.

## Resource Rules

1. Keep local stack usage small. Anything meaningfully larger than 256 bytes should be justified.
2. Avoid repeated heap churn in loops. Allocate once in `onEnter()`, reuse, and free in `onExit()`.
3. Large constant tables should be `static const` so they live in flash, not DRAM.
4. Avoid `std::string` and Arduino `String` in hot paths. Prefer `string_view`, `char[]`, and `snprintf`.
5. All user-facing UI strings must use `tr(STR_*)`. Logs may be hardcoded.
6. Prefer `constexpr` for compile-time constants.
7. Reserve `std::vector` capacity before push loops.
8. Debounce persistent writes. Do not write progress on every page turn.
9. `new` is not nothrow on ESP32. With exceptions disabled, bare `new` calls `abort()` on allocation failure instead of returning `nullptr`. Use `new (std::nothrow)` or `makeUniqueNoThrow<T>()` from `lib/Memory/Memory.h` for fallible allocations.
10. Prefer `makeUniqueNoThrow<T>()` / `makeUniqueNoThrow<T[]>()` for owned heap allocations so cleanup is automatic on early returns.
11. Use raw `malloc` or `new (std::nothrow)` only when a C or SDK API takes ownership; add a short comment explaining that ownership transfer.
12. Use PSRAM deliberately for large, non-DMA buffers. Keep task stacks, ISR data, DMA buffers, and latency-sensitive state in internal RAM, and handle PSRAM allocation failure.

## HAL And Platform Rules

- Use HAL classes, not SDK classes, in app code.
- File I/O uses `FsFile`, not Arduino `File`.
- Always close files explicitly.
- Use `MappedInputManager::Button::*` enums for button logic.

## C++ / Embedded Gotchas

- `string_view::data()` is not null-terminated. Do not pass it directly to C APIs.
- ISR handlers need `IRAM_ATTR`, and ISR-read data must be in DRAM, not flash-only storage.
- Never call `xSemaphoreTake()` from an ISR. Use ISR-safe give APIs.
- Do not cast unaligned `uint8_t*` data to wider pointer types. Use `memcpy`.
- No exceptions. No `abort()`. Log before returning failure.
- Avoid `std::function` in hot paths and library code; prefer function pointers or a small context/callback struct.
- Keep template use deliberate. If a template is needed in shared code, consider explicit instantiation in a `.cpp` file to avoid repeated binary growth.

## Error Handling

- Prefer `LOG_ERR(...)` plus `return false` for recoverable failures.
- Prefer `LOG_ERR(...)` plus a known fallback when the app can continue safely.
- Use `assert(false)` only for truly impossible fatal states.
- Use `ESP.restart()` only for intentional recovery flows, such as completing OTA.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dito94/CrossDiTo](https://github.com/dito94/CrossDiTo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
