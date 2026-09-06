---
trigger: always_on
description: Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.
---

# CLAUDE.md

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

---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Philosophy

### Core Mission
**Be the standard library QuickJS deserves** by providing WHATWG-spec'd web APIs and Deno/Bun-like runtime APIs with a coherent, documented JS surface over native bindings.

### API Design Principles

1. **Prefer Standards Over Custom APIs**
   - Priority: WHATWG > Browser > Bun > Node > Deno
   - Avoid "qjs-modules-isms" (custom APIs that lock users in)
   - Scripts from browser/Node/Deno/Bun should run with minimal changes

2. **Prefer JS-Idiomatic APIs Over C++ API Parity**
   - When binding C++ containers, prefer plain JS arrays with GC over strict API reproduction
   - Use JS-native patterns: arrays, iterators, `for...of`, spread, indexing, length
   - Avoid verbose container APIs (`.get()`, `.delete()`, `.size()`) when plain arrays are cleaner
   - If C++-style containers are needed, enhance them with JS-idiomatic extensions (Symbol.iterator, indexing)

3. **Internal vs Public APIs**
   - Modules like `deep`, `predicate`, `pointer`, `misc` are internal implementation details
   - Don't add new custom public APIs unless absolutely necessary
   - Document rationale for any custom API added

## What this is

A collection of **native C modules for QuickJS** (`quickjs-*.c`/`.h` bindings, e.g. `stream`,
`xml`, `deep`, `dom`, `json`), plus JS-side wrappers and helpers in `lib/*.js`. Built via CMake;
tests live in `tests/test_*.js` and run under `qjs`/`qjsm`, wired up as CTest cases
(`add_test` in `CMakeLists.txt`).

## Recent Work (August 2026)

### Documentation Restructuring (Completed)
- **Reorganized `doc/` folder** into logical subdirectories:
  - `doc/native/` - C native modules (32 modules + README)
  - `doc/js/` - JavaScript modules (46 modules + README)
  - `doc/` - General documentation (README, grammar, buffer, readline, api-compatibility)
- **Created comprehensive READMEs** for both subdirectories explaining structure and classification
- **Updated all references** throughout codebase to new paths

### Module Classification System (Completed)
All modules classified into four categories:

1. **Native Modules** (32 C bindings in `doc/native/`):
   - Direct C implementations exposed to JavaScript
   - Examples: blob, stream, dom, fs, process, child-process, sockets
   - Document the JS API exposed by C bindings
   - **NO references to `lib/*.js` files**

2. **JavaScript Polyfills** (15 in `doc/js/`):
   - Standalone JS implementations of standard APIs
   - No native imports, work in other runtimes
   - Examples: deep.js, pointer.js, predicate.js, xml.js, misc.js, stream.js, events.js, abort.js

3. **JavaScript Wrappers** (18 in `doc/js/`):
   - Wrap native modules to provide higher-level APIs
   - Examples: fs.js, process.js, console.js, assert.js, streams.js


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rsenn/qjs-modules](https://github.com/rsenn/qjs-modules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
