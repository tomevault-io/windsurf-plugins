---
trigger: always_on
description: clice is a next-generation C++ language server (LSP) built on LLVM/Clang, targeting modern C++ (C++20/23). It uses a multi-process architecture with a master server coordinating stateless and stateful workers.
---

# clice — Project Guide

## Project Overview

clice is a next-generation C++ language server (LSP) built on LLVM/Clang, targeting modern C++ (C++20/23). It uses a multi-process architecture with a master server coordinating stateless and stateful workers.

## Core Correction Patterns — Lessons from Past Interactions

The following patterns were extracted from extensive real-world collaboration. These are recurring mistakes that MUST be avoided. Read them carefully — they represent hard-won lessons, not hypothetical concerns.

### Pattern 1: Misjudging Real-World Priorities

AI tends to optimize whatever metric looks most impressive, rather than what actually matters in the user's real scenario.

**Example**: During performance optimization, the AI proudly reported "hot cache is 4-7x faster!" — but the function in question runs at LSP server startup, which is ALWAYS a cold start. Optimizing hot cache was completely meaningless.

**Rule**: Before optimizing or analyzing anything, first understand the REAL usage scenario. Ask yourself: "When does this code actually run? What does the user actually experience?" Do not chase metrics that look good on paper but are irrelevant in practice.

### Pattern 2: Pushing Without Local Verification

The most common and most damaging pattern. AI proposes a fix, pushes it immediately, CI fails, then another fix, push, fail again — wasting CI cycles and the user's time.

**Rule**: NEVER push code that you haven't verified locally. Before every push:

- Build locally with the same configuration CI uses.
- Run the relevant tests locally and confirm they pass.
- If you cannot reproduce the CI environment locally, say so — do not just "try and see."
- "It compiles" is NOT sufficient. Tests must pass.

### Pattern 3: Superficial Refactoring

When asked to refactor, AI tends to do mechanical code movement (copy functions from A to B) without understanding the deeper design intent (ownership, responsibility boundaries, API cleanliness).

**Example**: When splitting `MasterServer` into `Workspace` and `Session`, the AI moved functions but kept ugly APIs like `f(path_id, sessions_map)` instead of the clean `f(Session&)` that the refactoring was meant to achieve.

**Rule**: When refactoring, understand the WHY. Ask: "What design problem is this refactoring solving?" If you're just moving code around without improving the abstractions, you're not refactoring — you're rearranging deck chairs.

### Pattern 4: Fixing Only the Immediate Instance, Not the Pattern

When given a cleanup instruction, AI applies it to the single file or function currently being discussed, ignoring all other occurrences in the project.

**Example**: User says "remove decorative `===` comment separators." AI removes them from `workspace.cpp` only. User has to say: "The other files too!"

**Rule**: When given a cleanup or style instruction, apply it project-wide. Use `Grep` to find ALL occurrences and fix them all in one pass. Think: "Where else does this pattern appear?"

### Pattern 5: Never Skip, Disable, or Work Around Failing Tests

When stuck on a difficult bug (especially flaky CI, race conditions, platform-specific issues), AI may propose marking tests as `continue-on-error`, skipping them, or adding `expected-failure` annotations to make CI green.

**Rule**: This is ABSOLUTELY FORBIDDEN. If a test fails, fix the root cause. There are ZERO exceptions. Skipping a test to make CI green is not "fixing" — it is hiding a bug. If you ever find yourself thinking "maybe we should just skip this test," stop and reconsider your approach entirely.

### Pattern 6: Excessive Confirmation Seeking vs. Premature Execution

AI oscillates between two extremes: asking "should I do X?" for every trivial decision, or silently executing major changes without confirmation.

**Rule**: Calibrate based on reversibility and impact:

- **Small, reversible changes** (formatting, renaming a local variable, adding a test): just do it.
- **Architecture decisions, API changes, large refactors**: propose the plan first, wait for confirmation.
- **Pushing to remote, creating PRs, modifying CI**: always confirm.
- When the user says "go ahead" or "do it," execute fully without asking again mid-way.

## Code Reuse & Understanding Before Implementation

**This is the single most important rule in this project.** Before writing ANY new code, you MUST thoroughly read and understand the existing codebase first. This project has a rich set of utilities, abstractions, and patterns already in place — duplicating them wastes effort and creates maintenance burden.

Concrete requirements:

1. **Read before you write.** Before implementing a feature or fix, explore the relevant modules in `src/`. Search for existing helpers, utilities, and patterns that solve the same or similar problems. Use `Grep`, `Glob`, and `Agent` tools to investigate thoroughly — do not assume something doesn't exist just because you haven't seen it yet.

2. **Reuse existing infrastructure.** This project already has:
   - A `Lexer` class (`src/syntax/lexer.h`) — do not hand-write token scanning logic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clice-io/clice](https://github.com/clice-io/clice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
