---
trigger: always_on
description: Trust this document first.
---

# Copilot Cloud Agent Onboarding

Trust this document first.
Only search the repository when information here is missing or proves incorrect.

## COWEL Language Reference

For a concise agent-friendly summary of COWEL syntax, types, directives, and content policies,
see [`.github/lang-summary.md`](lang-summary.md).

Whenever a language change is made to the COWEL language
(new syntax, changed semantics,
new or removed builtin directives that are documented in these Markdown files,
type system changes, etc.),
both this file and `lang-summary.md` must be updated to reflect the change.

## Repository Summary

- `cowel` is a C++23 + Node.js project for **Compact Web Language (COWEL)**,
  a TeX-like markup language that compiles to HTML.
- The native product is a CLI (`cowel-cli`) and static library (`cowel`).
- The npm product is a Node CLI backed by a WASM build (`cowel-npm` target, outputs to `build/npm`).
- Main languages:
  - C++ (core compiler)
  - TypeScript/JavaScript (npm CLI/tests),
  - COWEL documents (`.cow` or `.cowel`),
  - Python build helpers
- Approximate size: medium-large monorepo with embedded/third-party content
  (`third_party/boost`, `ulight/`, generated build trees).

## Documentation Style

- The project uses semantic line breaks for comments and documentation:
  https://sembr.org/
- When editing Markdown, prose comments, or long documentation strings,
  write one semantic unit per line and reflow changed prose to this style.
- Apply semantic line breaks as a transform,
  not only for new text but also for touched surrounding prose.

## High-Value Layout (Start Here)

- Root build system: `CMakeLists.txt`
- Bindings test/build reference: `bindings/README.md`
- Core C++ headers: `engine/include/cowel/`
- Core C++ sources: `engine/src/`
- C++ tests: `engine/test/src/`
- Native CLI wrapper: `bindings/native/src/`
- Node wrapper TS sources: `bindings/node/src/`
- Node wrapper TS tests: `bindings/node/test/`
- Utility scripts: `tools/`
  - includes `coverage-llvm.sh` for LLVM source-based coverage
- Docs + golden sample I/O: `docs/index.cow` and `docs/index.html`
- VS Code extension + TextMate grammar tests: `editor/vscode/`
- CI workflows:
  - `.github/workflows/cmake-multi-platform.yml`
  - `.github/workflows/clang-format.yml`
  - `.github/workflows/textmate-test.yml`
  - `.github/workflows/coverage.yml` (LLVM coverage via clang-20, uploads to Codecov)

Important dependency facts not obvious from tree:
- Native configure requires ICU (`find_package(ICU COMPONENTS data i18n uc REQUIRED)`).
- Native configure requires Python 3 (`find_package(Python3 REQUIRED)`),
  used to embed assets (`tools/file-to-array.py`).
- If `third_party/boost` is missing, configure auto-clones Boost via `tools/boost-install.sh`.
- `ulight` is a git submodule and is built as part of the top-level CMake project.

## Toolchain Versions Validated Locally

- Node.js `v20.19.6`
- npm `10.8.2`
- CMake/CTest `4.3.2`
- Python `3.12.3`
- GCC detected by CMake: `13.3.0`

CI also uses:
- Node 20
- gcc-13 and clang-20 variants
- clang-format-20
- Emscripten toolchain for WASM path

## Always-Use Command Order (Native)

Run from repo root.

1. Bootstrap/preconditions

```bash
git submodule update --init --recursive
npm install --prefix bindings/node
```

2. Configure clean native build

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
```

3. Build native targets

```bash
cmake --build build --config Debug -j4
```

4. Run C++ tests

```bash
ctest --test-dir build --output-on-failure
```

5. Run CLI golden validation

```bash
./build/cowel-cli run docs/index.cow build/docs.actual.html
diff -u docs/index.html build/docs.actual.html
```

### Validated outcomes/timings

- Building before configure fails fast
  (`Error: .../build is not a directory`, ~0.14s).
  Always configure first.
- Configure succeeded (~11.8s).
- Full native build succeeded (~57.8s).
- Native tests succeeded: `332/332` passed
  (~47.8s wall clock; CTest real ~45.8s).
- CLI docs golden diff succeeded (no diff, ~0.27s).

## Node/TypeScript Validation

From `bindings/node/`:

```bash
npm ci
npm run build
npm run build:test
npx eslint src --max-warnings=0 --color
npm test
```

Validated outcomes/timings:
- `npm run build` succeeded (~0.82s)
- `npm run build:test` succeeded (~1.09s)
- ESLint command succeeded (~2.06s)
- `npm test` succeeded (`96` pass, `0` fail, ~0.30s)

## VS Code Grammar Validation (CI Parity)

From `editor/vscode/`:

```bash
npm install
npm test
```

Validated outcomes/timings:
- install succeeded (~3.55s)
- tests succeeded
  (`35` passed, `1` fixture marked "without expectations", ~1.17s)

## Formatting Gate (CI Parity)

CI command (run from repo root):

```bash
find engine/include engine/src bindings/native/src bindings/node/src/cpp \
  \( -name '*.cpp' -o -name '*.c' -o -name '*.hpp' -o -name '*.h' \) |
  xargs clang-format-20 --color=1 --dry-run --Werror
```

Validated outcome:
- command succeeds when `clang-format-20` is available (~1.19s).

## WASM/NPM CMake Path

CI uses Emscripten and builds target `cowel-npm`.
CI also builds target `cowel-lsp-wasm`.

Expected sequence:
1. Install and activate emsdk.
2. Configure with Emscripten toolchain file.
3. Build `cowel-npm` and `cowel-lsp-wasm` targets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eisenwave/cowel](https://github.com/eisenwave/cowel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
