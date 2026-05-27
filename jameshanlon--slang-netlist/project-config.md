---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

Configure using a CMake preset (see `CMakePresets.json` for options; `CMakeUserPresets.json` for local overrides):

```sh
cmake --preset macos-debug        # configure (macOS)
cmake --preset clang-debug        # configure (Linux, clang)
cmake --build build/macos-debug   # build
```

Python bindings are off by default (`option(ENABLE_PY_BINDINGS ... OFF)` in `CMakeLists.txt`); the `clang-debug`/`clang-release`/`gcc-debug`/`gcc-release`/`macos-debug`/`macos-release` presets turn them on. Pass `-DENABLE_PY_BINDINGS=ON` when configuring with any other preset.

Run all tests:

```sh
ctest --test-dir build/macos-debug
```

Run a single unit test (Catch2 supports `-k` for filtering):

```sh
./build/macos-debug/tests/unit/netlist_unittests -k "test name pattern"
```

Run only the Python driver tests:

```sh
ctest --test-dir build/macos-debug -R python-driver-tests
```

## Code Style

- Follow [LLVM Coding Standards](https://llvm.org/docs/CodingStandards.html) with these exceptions:
  - 80-column width
  - Functions, parameters, and local variables use lowerCamelCase (not UpperCase)
  - `#pragma once` instead of `#ifdef` guards
  - Exceptions are generally not permitted.
- Run `clang-format` with the project's local `.clang-format` settings before committing.
- Install pre-commit hooks (`pip install pre-commit && pre-commit install`); the hook runs `clang-format` automatically.
- Library code lives in the `slang::netlist` namespace; the reporting visitors in `include/report/` live in `slang::report`.
- Keep comments short and concise. Do not add commentary that is related to the
  process of development. Prefer high level explanations rather than specific
  references to the code structure and naming.
- Always comment public API functions/methods and classes with an appropriate
  docstring or doxygen syntax.
- Format Python docstrings with the triple-quotes on separate lines.

## Workflow

- When changes have been completed, review the patch in detail to look for
  opportunities for simplification.
- Do not sign commits with `Co-authored-by: ...` trailers.

## Architecture

Slang Netlist is a C++ library that builds a **dependency graph** (the "netlist") over an elaborated SystemVerilog AST provided by [slang](https://sv-lang.com). The graph captures source-level static connectivity at bit-level granularity.

Generated documentation lives in `docs/`: `user-guide.dox` covers CLI usage, `developer-guide.dox` covers internals, and `mainpage.dox` is the Doxygen entry point.

### Core Components

**Graph data structures** (`include/netlist/`):
- `DirectedGraph<NodeType, EdgeType>` — generic directed graph template
- `NetlistGraph` — specialization holding `NetlistNode`/`NetlistEdge`; the central artifact of the library
- `NetlistNode` — polymorphic base; subtypes: `Port` (I/O), `Variable` (wire/reg), `State` (sequential persistent value), `Assignment`, `Conditional`, `Case`, `Merge` (branch join), `Constant` (literal value driver). Nodes represent operations or state
- `NetlistEdge` — directed edge (producer→consumer) annotated with driven symbol, bit range, and `ast::EdgeKind` (clock sensitivity). Edges represent data dependencies

**Graph construction** (`source/`, `include/netlist/`):
- `NetlistBuilder` — main AST visitor (extends `slang::ast::ASTVisitor`). Composed of `NodeFactory`, `PortConnectionHandler`, `PendingRvalueQueue`, `CanonicalBodyResolver`, and a `BuildPipeline` that owns the four-phase orchestration
- `BuildPipeline` — orchestrates the four-phase build: (1) sequential AST traversal to create ports/variables/instances and collect deferred DFA blocks, (2) DFA dispatch over deferred blocks (parallel when `options.parallel`), (3) drain thread-local pending R-values, (4) resolve pending R-values into graph edges (parallel when above threshold)
- `DataFlowAnalysis` — extends `slang::analysis::AbstractFlowAnalysis`; computes **reaching definitions** (which nodes last wrote each bit range), unlike slang's `DefaultDFA` which only tracks whether ranges are driven. Handles procedural blocks (always/initial) including if/case branching, loop unrolling, and non-blocking assignments
- `NodeFactory` — centralizes node allocation; registers each new node with the `NetlistGraph` and (for value-bearing kinds) records the (symbol, bounds) → node mapping in the builder's `VariableTracker`
- `PortConnectionHandler` — handles port-connection wiring; owns the slice allocator and `CutRegistry` that propagates concat-shaped actuals' bit boundaries down to formal ports
- `PendingRvalueQueue` — accumulates deferred R-values during Phase 2 (thread-local per task to avoid contention), then resolves them into edges in Phase 4
- `CanonicalBodyResolver` — redirects driver queries for non-canonical instance bodies to their canonical counterparts, since slang's `AnalysisManager` stores drivers only against canonical bodies
- `ValueTracker` / `VariableTracker` — interval-map-based structures that track which netlist nodes drive which bit ranges of each symbol

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jameshanlon/slang-netlist](https://github.com/jameshanlon/slang-netlist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
