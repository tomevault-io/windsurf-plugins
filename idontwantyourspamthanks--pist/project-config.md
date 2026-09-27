---
trigger: always_on
description: Guidance for an AI agent (or any contributor) working in this repository. Read this first; it tells
---

# AGENTS.md

Guidance for an AI agent (or any contributor) working in this repository. Read this first; it tells
you what the project is, the rules you must not break, and where to find everything.

## What this is

**PiST** — an IDE for Atari ST 68000 assembly development, built with Qt 6 (Widgets). You write
assembly, build it with `vasmm68k_mot`, run it in the Hatari emulator, and debug it (breakpoints,
stepping, registers, memory, disassembly, watchpoints, stack, hardware registers) with the editor
following the program counter. C++17, CMake, GPL-2.0-or-later.

## Build and test

```sh
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
QT_QPA_PLATFORM=offscreen ctest --test-dir build --output-on-failure
```

- Parser/unit tests always run. Integration tests (`tst_gui`, `tst_remotecontrol`,
  `tst_emulatorhost`, `tst_hrdb`) run **offscreen** and `QSKIP` unless Hatari, `vasmm68k_mot` and a
  TOS ROM are present — set `PIST_TOS_DIR=<dir>` and put the tools on `PATH`; `PIST_REQUIRE_EMULATOR=1`
  makes a skip fail (CI uses this). `tst_hrdb` additionally needs `$PIST_HRDB_HATARI` naming the
  hrdb-main fork binary, which CI builds and supplies.
- Keep `QT_QPA_PLATFORM=offscreen` on every test run, including local ones. Tests can open modal
  dialogs; on a desktop session those are real windows that hang the run until someone clicks them
  away. Running a test binary directly without the variable is how that happens.
- To run the app: `./run.sh` (builds if needed, opens `demo/hello.s`). Needs a real display and, for
  a full debug session, Hatari + a TOS ROM.
- **Verify a change by running the specific test that covers it.** For debug-loop changes, prove it
  against a real emulator run, not a mock. The full suite must stay green.

## The rules you must not break

These are correctness requirements, learned from real failures (each is detailed, with evidence, in
`docs/ARCHITECTURE.md` "Invariants" and `docs/PLAN.md` §5/§11):

1. **Never patch, link, or write config for a third-party tool.** vasm and Hatari are driven as
   subprocesses via command-line arguments only. Never write a symbol sidecar next to the `.PRG`;
   never touch the user's `hatari.cfg`. This is a licensing *and* correctness constraint.
2. **The debug transport has a hard channel split:** stdin carries debugger commands (works while
   stopped); the control socket carries control commands (works only while running). Do not cross them.
3. **Frame debugger responses on the `> ` prompt, not on content**, on both stdout and stderr; a
   `> <cmd>` echo is not a prompt; drain the pipe with a `bytesAvailable()`/`waitForReadyRead()` loop.
4. **Arm breakpoints only after `info basepage` arrives** (`setLiveBases()` → `armBreakpoints()`);
   breakpoints are file:line, never raw addresses (GEMDOS relocates the program each run). Resume
   must not drop queued `b` commands — Continue is live at the entry stop, before arming finishes.
5. **Capability-probe Hatari by option name, never by version number.**
6. **Never emit `echo` into a Hatari script file** (aborts 2.6.1).
7. **Keep tests independent of a real display/emulator** (they must skip cleanly when those are absent).
8. **Never open a user's file with `Truncate` and check the byte count afterwards** — the file is
   already empty when the check fails. Write through `files::write()` (`src/support/FileWrite.h`),
   which stages, flushes, checks the device error and only then replaces. Do not delegate this to
   `QSaveFile::commit()`: on the Qt 6.8.1 the archives bundle it renames a truncated temporary file
   over the destination and returns true when the flush inside it fails.

When you change a verified fact, a launcher rule, or a phase scope, update `docs/PLAN.md`;
contradictions of the plan go in its §9 ledger rather than being silently absorbed.

## Where to find things

Start with **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** — it has the full module map, the
build/run/debug flows with the function names that drive them, the invariants, and how to extend the
code. Use this table for a quick lookup:

| I want to change… | Go to |
|---|---|
| The app shell, menus, docks, panels, layout | `src/ui/MainWindow.{h,cpp}` |
| A debug panel (registers, memory, stack, …) | the matching `src/ui/<Name>View.{h,cpp}` |
| The text editor or gutter/breakpoints | `src/editor/CodeEditor.{h,cpp}` |
| The sprite / image editor | `src/ui/ImageEditor.{h,cpp}` + `src/image/` |
| `.pim` format, palettes, ST codecs | `src/image/{ImageDocument,Palette,StFormats,Transform}.*` |
| Project files / Disk A / Disk B | `src/ui/FileBrowser.{h,cpp}` |
| Git status, commit, pull, push, blame | `src/git/GitService.{h,cpp}`, `src/ui/GitPanel.{h,cpp}`, blame lane in `src/editor/CodeEditor.{h,cpp}` |
| `.st` / `.msa` floppy images | `src/build/FloppyImage.{h,cpp}` |
| Syntax highlighting | `src/editor/AsmHighlighter.{h,cpp}` |
| The build pipeline / diagnostics | `src/build/BuildService.{h,cpp}` |
| PC ↔ source-line mapping | `src/build/{LineMap,LinkMap,ProgramLineMap}.{h,cpp}` |
| How any file is written to disk (atomic replace) | `src/support/FileWrite.{h,cpp}` (`files::write()`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [idontwantyourspamthanks/PiST](https://github.com/idontwantyourspamthanks/PiST) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
