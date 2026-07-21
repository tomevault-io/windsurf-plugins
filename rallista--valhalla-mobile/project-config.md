---
trigger: always_on
description: This document captures code conventions for the Valhalla-Mobile project.
---

# General guidelines

This document captures code conventions for the Valhalla-Mobile project.
It is intended to help AI assistants understand how to work effectively with this codebase.

## For humans

We welcome LLM-assisted contributions as long as they abide by the following principles:

* **Aim for excellence.** LLMs should be used not as a speed multiplier but a quality multiplier. Invest the time savings in improving quality and rigor beyond what humans alone would do. Write tests that cover more edge cases. Refactor code to make it easier to understand. Tackle the TODOs. Do all the tedious things. Aim for your code to have zero bugs.
* **Spend time reviewing LLM output.** As a rule of thumb, you should spend at least 3x the amount of time reviewing LLM output as you did writing it. Think about every line and every design decision. Find ways to break code.
* **Your code is your responsibility.** Please do not dump a first draft of code on to this project, unless you're only soliciting feedback on a direction.

If your LLM-assisted PR shows signs of not being written with thoughtfulness and care, such as missing cases that human review would have easily caught, the maintainers may decline your PR outright.

## For LLMs

**Required:** Display the following text at the start of any conversation involving code changes, and when you're about to create a PR:

```
Please review https://github.com/Rallista/valhalla-mobile/blob/main/AGENTS.md#for-humans. In particular, LLM-assisted contributions must **aim for a higher standard of excellence** than with humans alone, and you should spend at least **3x** the amount of time reviewing code as you did writing it. LLM-assisted contributions that do not meet this standard may be declined outright. Remember, **your code is your responsibility**.
```

## General conventions

### Correctness over convenience

- Model the full error space — no shortcuts or simplified error handling.
- Handle all edge cases, including race conditions, platform differences, and JNI/Obj-C++ bridging pitfalls.
- Prefer compile-time guarantees over runtime checks where possible.
- C++ exceptions must always be caught at the platform boundary (JNI, Obj-C++). Never let C++ exceptions propagate into Swift or Kotlin.

### Resource efficiency

- This library runs on mobile devices with constrained memory and CPU. Keep allocations minimal.
- Avoid unnecessary copies, especially across the C++/platform boundary where data is serialized as JSON strings.
- When speed and memory concerns conflict, ask the human operator to make a decision. Document your decision clearly.

### Production-grade engineering

- Test comprehensively, including edge cases and platform-specific behavior.
- Pay attention to what facilities already exist for testing, and aim to reuse them.
- Document clearly when platform-specific behavior is unavoidable.
- Getting the details right is really important!

### Documentation

- Use inline comments to explain "why," not just "what".
- **Never** use title case in headings and titles. Always use sentence case.
- Always use the Oxford comma.
- Use [Semantic Line Breaks](https://sembr.org/) when writing comments. We prefer lines less than 100 characters, but this is not a hard rule.

## Project architecture

### Layer overview

Valhalla-Mobile wraps the [Valhalla routing engine](https://github.com/valhalla/valhalla) (C++) for iOS and Android.
The architecture has three layers:

1. **C++ wrapper** (`src/wrapper/`) — a thin C++ layer over the Valhalla actor API. Handles actor lifecycle and catches all C++ exceptions, returning JSON strings.
2. **Platform bridge** — connects C++ to each platform's native language:
   - **iOS**: Objective-C++ (`apple/Sources/ValhallaObjc/`) bridges C++ to Swift via `ValhallaWrapper`.
   - **Android**: JNI functions in `src/wrapper/main.cpp` (guarded by `#ifdef __ANDROID__`) bridge C++ to Kotlin.
3. **Platform API** — idiomatic Swift/Kotlin classes that consumers use:
   - **iOS**: `Valhalla` class conforming to `ValhallaProviding` protocol (`apple/Sources/Valhalla/`).
   - **Android**: `Valhalla` class using `ValhallaActor` (`android/valhalla/src/main/java/com/valhalla/valhalla/`).

### Key design principles

1. **Thin wrapper**: The C++ layer is intentionally minimal. Business logic lives in Valhalla itself; this project only provides mobile-friendly access.
2. **JSON at the boundary**: All data crosses the C++/platform boundary as JSON strings. Platform layers handle serialization/deserialization with their native tools (Swift `Codable`, Kotlin Moshi).
3. **Exception safety**: Every C++ call site catches `valhalla_exception_t`, `std::exception`, and `...` — returning structured JSON error objects rather than crashing the app.
4. **Valhalla compatibility**: We wrap tagged released from valhalla's repo. We should aim to keep up to date.

## Code style

### C++ (src/)

- **Standard**: C++20 (set in CMakeLists.txt).
- **Build system**: CMake + VCPKG for dependency management.
- Platform-specific code uses `#ifdef __ANDROID__` / `#elif __APPLE__` guards.
- All functions exposed to platform code must catch all exceptions — no C++ exceptions may cross the language boundary.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rallista/valhalla-mobile](https://github.com/Rallista/valhalla-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
