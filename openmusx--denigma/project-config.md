---
trigger: always_on
description: Repository guidance for agents working in `denigma`.
---

# AGENTS.md

Repository guidance for agents working in `denigma`.

## Skills

This repository's conventions live in `.agents/skills/<name>/SKILL.md`. They are not
stored under any single agent's own directory, so read them from `.agents/skills`
regardless of which agent you are. A skill overrides general practice for the work it
covers, so read the relevant one before starting that kind of work rather than after.

- `accidental-style` — rendering accidentals in names: Unicode for exported content, ASCII for output filenames and log messages.
- `classifier-design` — creating or editing classifiers in `src/classify`, or shared classification helpers used by exporters.
- `code-comments` — writing or revising a Doxygen comment in a public header, an implementation comment anywhere in `src`, or reviewing the comments in a change.
- `denigma-test-harness` — building, running tests, interpreting test failures, or choosing the test executable's working directory.
- `enum-mappings` — adding or modifying enum conversions in any `<exporter>_enums.cpp`.
- `marking-categories` — reading any field that both a musx `MarkingCategory` and a `TextExpressionDef`/`ShapeExpressionDef` carry (positioning, fonts, `useCategoryPos`/`useCategoryFonts`).
- `mnx-optional-export` — exporting MNX `OPTIONAL` and `OPTIONAL_WITH_DEFAULT` properties, with a preference for omitting values when possible.
- `nested-namespaces` — any C++ namespace declaration.
- `optional-usage` — `std::optional` fields or return values, especially for enum or bool state.
- `string-lookups` — hard-coded string-to-value lookups or dispatch, especially repeated literal comparisons.
- `windows-minmax` — any call to `std::min`, `std::max`, `std::clamp`, or `numeric_limits<T>::min`/`max`.

## Purpose

`denigma` is a C++23 CMake project that converts Finale MUSX content into Enigma XML and related formats.
The repository builds a CLI plus reusable libraries for classification, massage, export, and format conversion.

## Project Layout

- `src/core` contains shared domain code and the main library entry points.
- `src/classify` contains clef, articulation, dynamic, expression, and jump classification helpers.
- `src/formats/enigmaxml`, `src/formats/mnx`, `src/formats/mss`, and `src/formats/svg` contain the format-specific converters.
- `src/massage` contains MusicXML transformation helpers.
- `src/export` contains export-related code shared by tests and production targets.
- `src/io` and `src/utils` contain lower-level helpers.
- `src/wasm` contains the WebAssembly C ABI wrapper built by the `denigma_wasm` target.
- `tests` contains the GoogleTest suite and fixture data.
- `tests/data/inputs` contains checked-in input fixtures.
- `tests/data/inputs/reference` contains checked-in expected-output fixtures.
- `tests/data/outputs` contains generated output artifacts and should be treated as disposable unless a test update explicitly requires it.

## Build Rules

- Use an out-of-source build only. The top-level `CMakeLists.txt` rejects in-source builds.
- The normal build entry point is `build.cmake`:
  - `cmake -P build.cmake`
  - `./build.cmake`
- To clean the build tree:
  - `cmake -P build.cmake -- clean`
  - `./build.cmake -- clean`
- The build downloads third-party dependencies through `FetchContent`, including `pugixml`, `nlohmann_json`, `zlib`, and `googletest`.
- If you need a local MUSX DOM checkout, set `MUSX_LOCAL_PATH` in CMake rather than editing dependency logic.
- The WebAssembly module (`src/wasm`, target `denigma_wasm`, option `denigma_BUILD_WASM`) is built with Emscripten:
  - `emcmake cmake -S . -B build-wasm -DCMAKE_BUILD_TYPE=MinSizeRel -DDENIGMA_CXX_STANDARD=20`
  - `cmake --build build-wasm --target denigma_wasm`
  - `node tests/wasm/smoke.mjs build-wasm/wasm/denigma.js build-wasm/wasm/denigma.wasm`
- Always name the `denigma_wasm` target for that build. The `all` target also compiles the text-measuring converters and `denigma_textmetrics`, which the module does not link and which do not compile under Emscripten.
- The exported function list in `src/wasm/CMakeLists.txt` and the C ABI in `src/wasm/denigma_wasm.cpp` are the contract with `denigma-online` and `viritura`, which consume the module built from a pinned Denigma commit. Changing either changes those consumers.

## Test Rules

- Build the test target through CMake, then run the test binary from `tests/data`.
- The test executable is `denigma_tests` and is emitted under `build/tests`.
- Preferred test flow:
  - `cmake --build build`
  - `../../build/tests/denigma_tests`
- Run `../../build/tests/denigma_tests` with `tests/data` as the working directory.
- The GoogleTest binary expects the current working directory basename to be `data`; running it from the repository root causes broad false failures.
- `ctest --test-dir build` may report no registered tests even when `build/tests/denigma_tests` exists.
- `ctest --test-dir build/tests` can be used only after confirming the discovered tests have the correct `WORKING_DIRECTORY` registration.
- When investigating a focused regression, prefer a narrow GoogleTest filter on the direct executable before a full suite.
- Use `ctest -R ...` only after confirming CTest registration and working directories are correct.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openmusx/denigma](https://github.com/openmusx/denigma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
