---
trigger: always_on
description: Casso is a 6502 CPU emulator, assembler, and Apple II platform emulator in C++.
---

# Copilot Instructions for Casso

## Project Overview

Casso is a 6502 CPU emulator, assembler, and Apple II platform emulator in C++.
The solution has five projects:

- **CassoCore**: Static library containing CPU logic, assembler, parser, opcode table
- **CassoEmuCore**: Static library containing Apple II devices, video modes, audio generator
- **Casso**: Win32 GUI application (Apple II emulator, links CassoCore and CassoEmuCore)
- **CassoCli**: Console application (AS65-compatible assembler CLI, links CassoCore)
- **UnitTest**: DynamicLibrary (Microsoft Native CppUnitTest, links CassoCore and CassoEmuCore)

### Architecture: Thin Exe, Rich Testable Core (NON-NEGOTIABLE)

New code goes in the **core libraries** (`CassoCore` / `CassoEmuCore`), NOT an exe. An exe is an empty shell over a core entry function, `CliMain` for `CassoCli`, its GUI equivalent for `Casso`. All emulation, parsing, rendering, device models, persistence, and lifecycle/orchestration logic lives in the core static libs, which both the exe **and** the `UnitTest` project link. This is the whole reason the split exists: so essentially everything is unit-testable and mockable.

- **UT-reachable litmus**: before placing code, ask "can the `UnitTest` project link and exercise this?" If a piece of logic can only be tested by running the exe, it is in the wrong place or the wrong shape (entangled with an `HWND`, device context, COM apartment, or menu id). Factor the logic into core behind data-in/data-out functions or interface seams.
- **The split is testability, NOT a platform boundary.** "Does this call a platform API?" is the wrong question and never justifies putting code in an exe. Calling Win32 is not a reason to live there: file I/O behind an interface seam, a registry read, a clipboard round-trip, a WIC image codec are all drivable by a test, so all belong in core.
- **What actually stays in an exe** is only what cannot exist without the process. For a console app that is `main` and nothing else, doing no more than calling a core entry function and returning what it returns. For a GUI app it is that plus the `HWND`, its message pump, and the device objects. `CassoCli.exe` is the worked example: 3,639 lines to 57.
- **The existing `Casso` exe has accreted logic that belongs in core** (e.g. `EmulatorShell`). That is debt to be extracted, NEVER a template; do not imitate it. Read exe files only to find wiring points (where objects are constructed/owned, machine build/teardown hooks, menu dispatch), never as a structural template for new logic.

See the Constitution's Principle VI (Thin Executable, Testable Core) and Principle II (Testing Discipline).

## C++ Specific Guidelines

### Precompiled Headers
- Every `.cpp` file MUST include `"Pch.h"` as its **first** `#include`
- **NEVER** use angle-bracket includes (`<header>`) anywhere except `Pch.h` or a library project's umbrella header (currently only `Dxui.h`)
- All system headers and STL headers belong in `Pch.h`
- Individual `.cpp` and `.h` files use only quoted includes (`"header.h"`) for project headers

### Code Style
- Use spaces for indentation (match existing code style)
- **NEVER** break existing column alignment in declarations
- **ALWAYS** preserve exact indentation when replacing code
- Keep functions focused and short: ideally under ~50 lines
- Each function should have a single clear purpose
- Braces always required, even for single-statement `if`/`while`/`for`/`switch`
- No comma-separated variable declarations
- Prefer in-class member initialization (`.h`) over constructor initializer lists (`.cpp`)
- **Function-call/declaration spacing.** Space before non-empty parens
  (`fn (arg)`, `MyClass::Method (a, b)`); **NO** space before empty
  parens (`fn()`, `obj.GetThing()`). Never `fn ()`. Applies equally to
  declarations, definitions, calls, member access, and method calls in
  test bodies. Run `rg -n '\w \(\)' Casso/ CassoCore/ CassoEmuCore/ CassoCli/ UnitTest/`
  on any new or merged code before committing, should return zero hits
  in lines you authored or merged.
- **Cast spacing.** Space after a C-style cast:
  `(float) std::numbers::pi`, not `(float)std::numbers::pi`. Same for
  `(int) value`, `(Word) addr`, etc.
- File-scope statics use Hungarian: `s_<typePrefix><Name>`. Type prefixes:
  `k` = constant, `psz` = null-terminated string ptr (narrow OR wide),
  `ch` = char (narrow OR wide), no special wide marker. E.g.
  `s_kpszHost` (LPCWSTR), `s_kchBullet` (wchar_t),
  `s_kRomCatalog` (constant array).
  The leading `s_` says **file-scope static** and nothing else, a class member
  or a function-local drops it and keeps the rest (`kPadDip`, `kpszTitle`), so
  the prefix alone tells you which you are reading. Whether a constant should
  be file-scope at all is decided by "Where a file-local constant goes" below;
  being constant is not the test.
- **No anonymous namespaces.** NEVER use `namespace {}`. Put file-local
  helpers as class `static` members, not free functions. More broadly,
  strongly prefer class members over free/global functions; a free function
  needs a very convincing justification.
- **Where a file-local *type* goes**, by what it is:
  1. **A class, anything with methods** → its own `.h` / `.cpp` pair, one

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [relmer/Casso](https://github.com/relmer/Casso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
