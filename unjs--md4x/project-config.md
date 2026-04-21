---
trigger: always_on
description: > **Always keep this file (`AGENTS.md`) and referenced `docs/*.md` files updated when making changes to the project.**
---

# MD4X

> **Always keep this file (`AGENTS.md`) and referenced `docs/*.md` files updated when making changes to the project.**
> **Always update `CHANGELOG.md` when adding removing user-facing features, APIs, build targets, CLI options, or library behavior.**

> C Markdown parser library (fork of [mity/md4c](https://github.com/mity/md4c))

- **Version:** 0.5.2 (next: 0.5.3 WIP)
- **License:** MIT
- **Language:** C (C89/C99 compatible)
- **Spec:** CommonMark 0.31.2
- **Build:** Zig (`zig build`)
- **JS Runtime:** Bun (do **not** use npm, pnpm, yarn, or npx — use `bun`/`bunx` exclusively)
- **Formatting:** Always run `bun fmt` after finishing code changes

## Project Structure

```
src/
  md4x.c              # Core parser (~6500 LoC)
  md4x.h              # Parser public API
  entity.c             # HTML entity lookup table (generated)
  entity.h             # Entity header
  md4x-wasm.c          # WASM exports (alloc/free + renderer wrappers)
  md4x-napi.c          # Node.js NAPI addon (module registration + renderer wrappers)
  renderers/
    md4x-props.h       # Shared component property parser (header-only)
    md4x-json.h        # Shared JSON writer + YAML-to-JSON helpers (header-only)
    md4x-html.c        # HTML renderer library (~500 LoC)
    md4x-html.h        # HTML renderer public API
    md4x-ast.c        # AST renderer library (~530 LoC)
    md4x-ast.h        # AST renderer public API
    md4x-ansi.c        # ANSI terminal renderer library (~450 LoC)
    md4x-ansi.h        # ANSI renderer public API
    md4x-meta.c        # Meta renderer library (~300 LoC)
    md4x-meta.h        # Meta renderer public API
    md4x-text.c        # Plain text renderer library (~350 LoC)
    md4x-text.h        # Plain text renderer public API
    md4x-markdown.c    # Markdown renderer library (~820 LoC)
    md4x-markdown.h    # Markdown renderer public API
    md4x-heal.c        # Markdown heal/completion utility (~600 LoC)
    md4x-heal.h        # Heal utility public API
  cli/
    md4x-cli.c           # CLI utility (multi-format: html, text, json, ansi, markdown, heal)
    cmdline.c            # Command-line parser (from c-reusables)
    cmdline.h            # Command-line parser API
    md4x.1               # Man page
packages/md4x/           # npm package
  package.json           # Package manifest (name: md4x)
  README.md              # Package README
  LICENSE.md             # MIT license
  build/
    md4x.wasm            # Prebuilt WASM binary
    md4x.*.node          # Prebuilt NAPI binaries (per-platform)
  lib/
    wasm.mjs             # JS entrypoint for WASM (async API, ESM)
    wasm.d.mts           # TypeScript declarations for WASM API
    napi.mjs             # JS entrypoint for NAPI (sync API, ESM)
    napi.d.mts           # TypeScript declarations for NAPI API
    types.d.ts           # Shared TypeScript types (ComarkTree, ComarkNode, ComarkElement, etc.)
  test/
    _suite.mjs           # Shared test suite (vitest, used by both NAPI and WASM tests)
    napi.test.mjs        # NAPI binding tests
    wasm.test.mjs        # WASM binding tests
  bench/
    _fixtures.mjs        # Benchmark fixture strings (small, medium, large)
    index.mjs            # Benchmark runner (mitata, compares napi/wasm/md4w/markdown-it)
test/
  spec.txt             # CommonMark 0.31.2 spec tests
  spec-*.txt           # Extension-specific tests (tables, strikethrough, frontmatter, etc.)
  regressions.txt      # Bug regression tests
  coverage.txt         # Code coverage tests
  run-testsuite.py     # Individual test suite runner
  pathological-tests.py # Stress tests for DoS resistance
  prog.py              # Program execution wrapper
  normalize.py         # HTML normalization for comparison
  fuzzers/             # LibFuzzer harnesses (html, ast, ansi, text, meta, heal)
    build.sh           # Build script for all fuzzers (clang + sanitizers)
    seed-corpus/       # Seed inputs (commonmark, gfm, frontmatter, components, etc.)
scripts/
  run-tests.ts            # Main test runner (runs all suites)
  build-entity-map.ts     # Generates entity.c from WHATWG spec
  build-folding-map.ts    # Unicode case folding map generator
  build-punct-map.ts      # Punctuation character map generator
  build-whitespace-map.ts # Whitespace classification generator
  _unicode-map.ts         # Shared helper for punct/whitespace map generators
  coverity.sh             # Coverity Scan integration
  unicode/                # Unicode data files (CaseFolding.txt, DerivedGeneralCategory.txt)
website/                 # Docs + playground (Vite + Vue)
  package.json           # Website package manifest (md4x-demo)
  vite.config.mjs        # Vite config (base: /md4x/)
  index.html             # HTML entry
  index.ts               # App entry point
  index.css              # Styles
  App.vue                # Root component with navigation
  router.ts              # Vue Router config
  env.d.ts               # TypeScript env types
  components/
    TabSelect.vue        # Responsive select/button tab switcher
    AnsiTerminal.vue     # xterm.js-backed ANSI output view
    ComarkVueRenderer.vue # Comark AST -> Vue VNode renderer for playground

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unjs/md4x](https://github.com/unjs/md4x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
