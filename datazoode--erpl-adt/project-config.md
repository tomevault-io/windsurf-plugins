---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Commits and Pull Requests

Do NOT include `Co-Authored-By`, `🤖 Generated with Claude Code`, or any AI attribution lines in commit messages or pull request bodies.

## Project Overview

`erpl-adt` is a CLI and MCP server for the SAP ADT REST API — a single C++ binary that talks the same HTTP endpoints Eclipse ADT uses. It enables AI coding agents and human developers to search, read/write source code, run tests, manage transports, and more against ABAP systems. No Eclipse, no SAP NW RFC SDK, no JVM.

Part of the Datazoo ERPL family. Shares build conventions with flapi and library choices with erpl-web.

## Build Commands

```bash
make release          # Full release build (CMake + Ninja + vcpkg)
make test             # Run unit tests (Catch2, no SAP system needed)
make clean            # Remove the build directory
```

For faster rebuilds during development:
```bash
cmake --build build --target erpl_adt_tests   # Rebuild tests only
ctest --test-dir build --output-on-failure     # Run tests only
```

Build requires: CMake, Ninja, vcpkg (git submodule at `vcpkg/`). Checkout with `--recurse-submodules`.

## Architecture

```
main.cpp -> command_router -> {group handlers}
                                    |
         +---------+---------+------+------+---------+---------+
         v         v         v      v      v         v         v
      search    object    source  test   check   transport   mcp
      locking    ddic    packages abapgit activation  deploy_workflow
         |         |         |      |      |         |         |
         +---------+---------+------+------+---------+---------+
                                    v
                             i_adt_session <-- adt_session (cpp-httplib)
                                    |
                             i_xml_codec  <-- xml_codec   (tinyxml2)
```

All arrows point downward. No cycles. Every horizontal boundary is a pure abstract interface (abstract base class).

**Directory decomposition:** `include/erpl_adt/{core,adt,cli,mcp,config,workflow}` mirrors `src/`. This reflects compilation firewall boundaries — changes to `adt/` internals don't force recompilation of `config/`.

**Key components:**
- `core/types.hpp` — 11 strong types: `PackageName`, `RepoUrl`, `BranchRef`, `RepoKey`, `SapClient`, `ObjectUri`, `ObjectType`, `TransportId`, `LockHandle`, `CheckVariant`, `SapLanguage`
- `core/result.hpp` — `Result<T,E>` discriminated union + `Error` struct with `ErrorCategory` for exit codes
- `adt/i_adt_session.hpp` — Abstract HTTP session (`Get`, `Post`, `Put`, `Delete`, stateful sessions)
- `adt/i_xml_codec.hpp` — Abstract XML codec (legacy, used only by deploy workflow)
- `adt/{search,object,locking,source,testing,checks,transport,ddic}.hpp` — Operation modules, stateless free functions taking `IAdtSession&`
- `adt/{packages,abapgit,activation}.hpp` — Deploy/bootstrap operations (`packages` uses `IXmlCodec`; `abapgit` and `activation` use shared async protocol contracts)
- `cli/command_router.hpp` — Two-level dispatch: `erpl-adt <group> <action> [args]`
- `cli/output_formatter.hpp` — Human-readable table and JSON output
- `mcp/mcp_server.hpp` — JSON-RPC 2.0 server over stdio (MCP 2024-11-05)
- `mcp/tool_registry.hpp` — Tool name → handler mapping
- `config/config_loader.hpp` — Merges CLI args (argparse) + YAML (yaml-cpp) into `AppConfig`
- `workflow/deploy_workflow.hpp` — Idempotent state machine: discover → package → clone → pull → activate
- `workflow/lock_workflow.hpp` — Lock transaction orchestration for CLI auto-lock flows
- `src/adt/protocol_kernel.hpp` — Shared 202+Location async polling contract
- `src/adt/{xml_utils,atom_parser}.hpp` — Shared parser primitives for namespaced XML/Atom feeds

**XML parsing strategy:** Operation modules parse XML with tinyxml2 and shared parser helpers (`xml_utils`, `atom_parser`) to reduce duplicated namespaced traversal logic. `IXmlCodec` is preserved for legacy deploy workflow paths.

**Testing:** Hand-written mocks in `test/mocks/` implementing the abstract interfaces. No mocking framework. Test fixtures in `test/testdata/` are real captured Eclipse ADT XML traffic.

## Design Constraints

These are hard requirements, not suggestions:

- **C++17.** `-Werror` enabled — treat all warnings as errors.
- **No raw `new`/`delete`** — `unique_ptr`/`shared_ptr` only. No global mutable state.
- **No exceptions for expected failures.** Use `Result<T,E>`. Exceptions only for programming errors.
- **Strong types** for all domain concepts. Private constructors + `Create()` factory returning `Result<T, string>`. No sentinel values — use `std::optional`.
- **Constructor injection** for all dependencies. Every component testable in isolation via mock collaborators.
- **RAII everywhere** — HTTP sessions, CSRF tokens, lock handles (`LockGuard`).
- **Const-correctness** — all non-mutating methods `const`, all non-modified params `const&` or `string_view`.
- **No implicit conversions, no C-style casts, no `void*`.**
- **Public interfaces minimal.** Internal helpers in anonymous namespaces, never in headers.

## Common Pitfalls


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataZooDE/erpl-adt](https://github.com/DataZooDE/erpl-adt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
