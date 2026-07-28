---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

Wowless is a headless World of Warcraft client Lua and FrameXML interpreter
intended for addon testing. It interprets WoW's client Lua code and XML UI
definitions without requiring the actual game client.

## Build Commands

Development is primarily done via Docker (devcontainer). Native builds are
also supported.

### Native Build

```sh
git submodule update --init --depth 1
cmake --preset default
cmake --build --preset default
```

Build output goes to `build/`.

### Running Wowless

```sh
# Build and run on retail WoW
bin/run.sh wow

# Run with an addon
bin/run.sh wow --addondir path/to/YourAddon

# Run directly after building
build/wowless run -p wow [options]
```

Products: see `data/products.yaml`

### Running Tests

```sh
cmake --build --preset default --target test
```

The test target rebuilds changed sources then runs `build/runtests`.
A successful run exits with code 0 and produces no output. Tests are also
run by the `build and test` pre-commit hook, so they execute automatically
on every commit.

Tests are in `spec/` directory and use luassert. Test specs are defined in
CMakeLists.txt around line 985. The test addon in `addon/Wowless/` contains
in-game tests that run inside the simulated WoW environment during `runtests`.

### Linting and Formatting

Pre-commit hooks handle linting. Key tools:

- **luacheck**: Lua linting (config in `.luacheckrc`)
- **stylua**: Lua formatting (config in `stylua.toml`)
- **clang-format**: C formatting (config in `.clang-format`)
- **yamlfmt**: YAML formatting (`build/yamlfmt`)

```sh
pre-commit run -a  # Run all checks
```

### C Code Style

clang-format handles most C formatting. One rule it cannot enforce:

- Multiline block comments must have the opening `/*` on its own line with no
  text after it:

  ```c
  /* wrong: text on the opening line
   * more text
   */

  /*
   * correct: /* is alone on its line
   * more text
   */
  ```

  Single-line comments (`/* text */`) are unaffected by this rule.

## Architecture

### Core Runtime (`wowless/`)

- `wowless.lua`: Entry point, parses CLI args and invokes runner
- `runner.lua`: Orchestrates WoW environment simulation (login, events, scripts)
- `modules.lua`: Dependency injection system loading modules from
  `wowless/modules/`
- `modules/`: Individual modules (api, events, loader, security, uiobjects,
  etc.)

### Module System

Modules are defined in `data/modules.yaml` with explicit dependencies. The
runtime loads them via topological sort. Each module in `wowless/modules/`
receives its dependencies as function arguments.

### Data Layer (`data/`)

YAML files define WoW API specifications, converted to Lua at build time:

- `data/products/<product>/`: Per-product API definitions (apis, events,
  uiobjects, etc.)
- `data/schemas/`: JSON-schema-like definitions for YAML validation
- `data/impl.yaml`, `data/uiobjectimpl.yaml`: Stub implementations

### Tools (`tools/`)

Build-time code generators:

- `gentest.lua`: Generates test addon code
- `prep.lua`: Preprocesses product data
- `docs.lua`: Documentation generator
- `yaml2lua.lua`/`lua2yaml.lua`: Format converters

### Test Addon (`addon/Wowless/`)

In-game test addon that runs within the simulated WoW environment to verify
API behavior.

### C Extensions

- `vendor/elune/`: Custom Lua 5.1 fork with WoW-specific extensions (taint
  tracking, security)
- `wowless/*.c`: Native Lua extensions (sqlite, mixin, bubblewrap, ext)

### External Data (`vendor/`)

- `vendor/dbdefs/`: WoW database definitions
- `vendor/tactless/`: CASC file extraction library

## Key Patterns

### Lua-to-C Compilation

The `lua2c()` CMake function compiles Lua modules into C for static linking.
This bundles all Lua code into the final executables.

### Product-Specific Data

Each WoW product (retail, classic, beta, etc.) has its own data directory
under `data/products/<product>/` with API definitions that may differ between
game versions.

### Security Model

Wowless implements WoW's taint/security system via elune extensions. Framework
code runs "secure" while addon code is "tainted".

### Userdata Objects

WoW exposes several userdata types (luaobjects, funtainers, uiobjects).
Key patterns:

- Use `newproxy(true)` to create userdata with custom metatables
- `__metatable = false` hides the real metatable from `getmetatable()`
- `bubblewrap()` wraps Lua functions to appear as C functions (fails
  `coroutine.create`)
- Methods should be readonly (error on assignment via `__newindex`)
- Custom fields stored in per-instance tables accessed via
  `__index`/`__newindex`
- `__tostring` format: `"TypeName: 0x..."` (use `tostring(table):sub(8)`
  for address)

### Test Addon Structure (`addon/Wowless/`)

The test addon runs inside the simulated WoW environment. Files load in
`.toc` order:

- `util.lua`: Assertion helpers (`assertEquals`, `check0`–`check7`, `match`,
  `retn`) stored on the addon table `G`
- `statemachine.lua`: `checkStateMachine(states, transitions, init)` for
  exhaustive state machine testing via BFS traversal of all edge combinations
- `init.lua`: Sets up `G.testsuite = {}` and `_G.assertEquals`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wowless/wowless](https://github.com/wowless/wowless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
