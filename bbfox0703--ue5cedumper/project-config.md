---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> For detailed specs, implementation history, and debugging notes, see the **[docs/](docs/)** directory.

-----

## Build & Deploy
- Before testing a change, confirm the binary under test was rebuilt from it (its timestamp or SHA): a failed build leaves the old DLL or `dist\` in place, and a stale binary passes silently.
- **Hand over an AOT-TRIMMED build, not the plain one.** `build.ps1` with no `-Mode` produces a
  self-contained **non-trimmed** exe (~107 MB); `-Mode Publish` produces the Native-AOT **trimmed**
  binary that ships (~54 MB). They are not the same program: reflection-shaped code — JSON without a
  source-generated context, MVVM bindings, `ComboBox.SelectedItem` bound to a boxed value, the
  reflection-based DataGrid column sort — compiles and runs fine untrimmed and fails **only** after
  trimming. **Every AOT bug in this repo's history was found by the maintainer re-compiling with AOT
  after being handed a non-trimmed build**, which costs a round trip every time.
  Use plain `build.ps1` / `-Target DLL` / `-Target Test` for fast iteration, then run
  `-Mode Publish` **before saying a UI change is ready to test**. It enters the VS DevShell itself
  (Native AOT links with MSVC's `link.exe`; without it ILCompiler dies with `exited with code 9009`,
  which reads like a broken toolchain rather than a missing environment).
  ⚠⚠ **ANY run that reaches the publish step overwrites `dist\UE5DumpUI.exe` with the
  non-trimmed exe** — `-Target UI`, `-Target Test` and a plain `build.ps1` alike; only
  `-Mode Publish` leaves an AOT-trimmed `dist\` (~54 MB against ~107 MB non-trimmed). The
  safest-looking command is the cheapest way to destroy the shippable binary.
  **After ANY build that touches the UI, re-run `-Mode Publish` and check the size/SHA before
  handing `dist\` over.** Let it bump the build number: the build number is the release number.

-----

## Code Changes
- When asked to refactor or rename modules/files, make actual code changes (move files, update imports, rename classes) — not just documentation updates. Confirm structural changes before proceeding to docs.

-----

## Debugging
- When fixing bugs, verify the fix against the actual memory layout or data structure rather than assuming. If the first fix doesn't work, re-examine fundamental assumptions about the data format before iterating.

-----

## Git Operations
- When creating PRs, check for branch divergence and resolve merge conflicts before attempting `gh pr create`. Run `git status` and `git log --oneline -5` first.
- ⚠ **Line endings are pinned by `.gitattributes` (`* text=auto eol=lf`), NOT by your git config
  — never "fix" them with `core.autocrlf`, which is machine-local (`true` at `--system` here) and
  does not travel between the two PCs.** Before the pin, `git checkout` silently rewrote an
  `i/lf w/lf` file to CRLF and left `git status` **CLEAN**, so `git checkout -- <file>` could not
  be trusted to revert a staged experiment byte-for-byte. `.gitattributes`' own header comments
  carry the rationale, the 2026-08-23 measurements and the `text=auto`-never-bare-`text` reason
  — read it before editing it.
- ⚠ **A whole-file diff on a small edit means the file was CORRUPTED, not reformatted.** Twice on
  2026-08-23 a patch script run through a shell heredoc had its `\\0` collapsed to `\0`, so Python
  wrote a **literal NUL byte** into a source file (`Mimic.cpp`) and then into `docs/todo.md`. Git
  and grep treat a NUL-bearing file as **binary**: the tells are `1483 insertions / 1470 deletions`
  on a 13-line edit, and `grep` answering `Binary file docs/todo.md matches`. **Check for NUL before
  blaming line endings**, and build a backslash numerically (`bytes([92])`) when patching through a
  heredoc.

-----

## Cheat Engine
- When working with CE Lua APIs, verify that functions/methods actually exist in the Cheat Engine Lua API before using them. Do not invent API calls.

-----

## Build & Dev Commands

### Unified Build Script

`build.ps1` handles VS DevShell setup, CMake configure, dotnet, and test execution, and it is the only builder that configures the tree or publishes `dist\`; bare `cmake` / `dotnet build` fail without the VS DevShell environment. For a verification-only DLL or C++ test build, use `py tools/verify/build_dll.py --targets <target...>`: it loads MSVC itself, never configures, and neither bumps `build_number.txt` nor touches `dist\`.

```bash
# Build everything (DLL + UI + Tests) — Release
powershell -NoProfile -ExecutionPolicy Bypass -File "D:\Github\UE5CEDumper\build.ps1"

# Build DLL + all 4 proxy DLLs (version/dinput8/dxgi/winmm — the injected artifacts)
powershell -NoProfile -ExecutionPolicy Bypass -File "D:\Github\UE5CEDumper\build.ps1" -Target DLL

# Build UI only
# ⚠⚠ ALSO republishes dist\ NON-TRIMMED — see ## Build & Deploy.
powershell -NoProfile -ExecutionPolicy Bypass -File "D:\Github\UE5CEDumper\build.ps1" -Target UI

# Run tests only
# ⚠ -Target Test does NOT compile the whole DLL. It builds 5 test executables, and
#   **10 of the 31** dll/src .cpp files reach a test target at all: dll_core_test #includes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbfox0703/UE5CEDumper](https://github.com/bbfox0703/UE5CEDumper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
