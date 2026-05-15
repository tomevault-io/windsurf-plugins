---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## Project Overview

mikrojs is an experimental JavaScript runtime for microcontrollers, powered by QuickJS-NG. It targets ESP32 via ESP-IDF and provides JS APIs for filesystem (LittleFS), GPIO, WiFi, HTTP, and timers. The core runtime is a standalone C++ library (`packages/@mikrojs/native/`) with zero ESP-IDF dependencies, enabling host-side testing. The codebase has two sides: C/C++ firmware (C++23, C11) and a Node.js/TypeScript tooling workspace at the root. Inspired by [txiki.js](https://github.com/saghul/txiki.js).

## Repository Layout

The project is a pnpm workspace (pnpm 10.30.1, Node >= 24). Key areas:

- **`packages/@mikrojs/quickjs/`** — QuickJS-NG engine wrapper package
  - `deps/quickjs/` — QuickJS-NG source (git submodule)
  - `quickjs.cmake` — Shared CMake module (dual-mode: CMake target + variable exports for ESP-IDF)
  - `postinstall.js` — Builds `qjsc` bytecode compiler at install time
  - `index.js` — Exports `cmakePath`, `includePath`, `qjscPath`
- **`packages/@mikrojs/native/`** — Standalone C++ library + Node-API addon
  - `include/mikrojs/` — Public headers (`mikrojs.h`, `platform.h`, `private.h`, etc.)
  - `src/` — Portable source files (runtime, modules, timers, REPL, etc.)
  - `runtime/` — TypeScript runtime modules (bundled to bytecode during CMake build)
  - `addon/` — Node-API addon (binding.cpp, runtime_wrap, platform_node)
  - `scripts/` — Build scripts for bytecode generation (esbuild bundling + qjsc compilation)
  - `test/` — Host-side tests (run via CMake/ctest)
  - `cmake.js` — Exports `cmakePath`, `includePath`, `srcPath`, `scriptsPath`, `runtimePath`, `bytecodeCmakePath`
- **`packages/@mikrojs/firmware/`** — ESP-IDF firmware package (`@mikrojs/firmware`)
  - `components/mikrojs/` — ESP-IDF adapter (compiles standalone lib + ESP-specific modules)
- **`esp32/`** — Thin consumer of `@mikrojs/firmware`; contains `CMakeLists.txt`, `package.json`, `.envrc`, `.gitignore`, `main/main.cpp`, and `test/`
- **`packages/mikrojs/`** — CLI tool (`mikro`/`mikrojs` commands) built with Ink/React for terminal UI
- **`packages/@mikrojs/`** — Shared packages: `analyze-imports`, `eslint-plugin`, `esptool`. Board and driver packages can be added here as workspace members.
- **`scripts/`** — Workspace package (`@repo/scripts`) for repo tooling (agent detection, etc.)
- **`packages/create-mikrojs/`** — Project scaffolding tool (`npm create mikrojs`)
- **`examples/`** — Example projects (`blank`, `blinky`, `neopixel`, `pwm-led`, `schema`, `sntp`, `uart`, `wifi-fetch`, `wifi-access-point`, and more)

Git submodule: `packages/@mikrojs/quickjs/deps/quickjs` (QuickJS-NG). Clone with `--recurse-submodules` or run `git submodule update --init --recursive`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikrojs/mikrojs](https://github.com/mikrojs/mikrojs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
