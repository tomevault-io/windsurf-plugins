---
trigger: always_on
description: Matching decompilation of Need for Speed Most Wanted 2005 (GameCube) targeting the USA Release build (`GOWE69`).
---

# Need for Speed Most Wanted Decompilation

Matching decompilation of Need for Speed Most Wanted 2005 (GameCube) targeting the USA Release build (`GOWE69`).
The goal is to produce C++ source that compiles to byte-identical and dwarf-identical object code against the
original retail binary using the ProDG GC 3.9.3 compiler, which is GCC 2.95-based under the hood.

## Build & Verify

```sh
python configure.py    # generate build.ninja + objdiff.json (run after config changes)
ninja all_source       # build all objects
ninja                  # build all objects, hash check and progress report
ninja baseline         # generates baseline report for regression checking
ninja changes          # check for regressions after code changes (empty = no regressions)
```

## Project Layout

```
src/                   C++ source and headers
include/               glibc headers
config/GOWE69/         Symbol addresses, section splits
  symbols.txt          Mangled symbol names -> addresses
  splits.txt           Memory layout / section boundaries
orig/                  Original game files
tools/                 Build system, agent tooling and other scripts
objdiff.json           Generated build/diff configuration
```

## Agent Tooling

## Sub-Agent Usage

Sub-agents are allowed only for **read-only exploration** tasks such as:

- searching the codebase for symbols, call sites, or include relationships
- inspecting decomp output, assembly, DWARF, PS2 dumps, or line mappings
- gathering context from Ghidra, `tools/decomp-workflow.py`, `lookup.py`, `decomp-diff.py`, or similar tools
- summarizing findings that help the main worker decide what to change

Sub-agents must **not** write or edit code files, headers, configs, or other repository files.
All persistent file changes, decomp implementations, scaffolding, and follow-up fixes must be
done by the main worker after reviewing the read-only findings.

## Forbidden Changes

Do **not** edit or otherwise touch the comparison and configuration inputs that define the
project's match metrics:

- `config/GOWE69/symbols.txt`
- `config/GOWE69/splits.txt`
- `configure.py`

Treat these files as read-only unless the user explicitly asks for a task that is specifically
about maintaining that infrastructure.

Do **not** try to cheat objdiff, progress, or match metrics in any way. The goal is to improve
the real decompilation output, not to manipulate the comparison setup, hide mismatches, or make
progress numbers look better without actually matching the original code.

**Never** copy, overwrite, or symlink a compiled source `.o` file into `build/GOWE69/obj/`.
The `obj/` directory contains the **original reference objects** extracted from the retail
binary by `dtk dol split`. Replacing them with your own compiled output will make objdiff
compare your code against itself, producing a false 100% match. If the `obj/` file is
accidentally corrupted, regenerate it with:

```sh
rm build/GOWE69/config.json
ninja build/GOWE69/config.json   # re-splits from the original ELF
```

### lookup.py — Symbol lookup from the debug dump

Query structs, enums, functions, globals, and typedefs directly from the pre-extracted
Dwarf dump.

See `.github/skills/lookup/SKILL.md` for the full workflow.

### lookup_address.py — Locate classes and inlines via debug line mapping

When you have a function's address and want to know which source file a class or inline
originates from, use this script against the compiler-generated debug line mapping:

See `.github/skills/line_lookup/SKILL.md` for the full workflow.

`line_lookup.py` now accepts both the original `0xADDR:` debug-line format and rebuilt
object exports written as bare `ADDR:` lines, so you can point it at
`symbols/debug_lines.txt` or at a rebuilt `debug_lines.txt` from
`tools/dwarf1_gcc_line_info.py`.

### elf_lookup.py — Resolve strings / rodata by virtual address

When you have a virtual address inside the original ELF and need to know which string or
rodata bytes live there, use:

```sh
python tools/elf_lookup.py 0x803E58F4
python tools/elf_lookup.py 0x803E58F4 --mode bytes --length 32
python tools/elf_lookup.py 0x002F1234 --game ps2
```

This is the preferred replacement for ad-hoc Python snippets that manually parse the ELF
to chase `@stringBase0` or other rodata/data references.

### code-style — Repo-local style guidance

When you are writing code, polishing code you already touched, or doing a style-review pass,
consult `.github/skills/code_style/SKILL.md` first. It captures repo-specific formatting and
cleanup rules, including jumbo include spacing, initializer-list comment markers, declaration
placement, pointer style, and how to keep style work safe in match-sensitive code.

Use `python tools/code_style.py audit --base origin/main` before a branch-wide style pass.
It classifies changed files, reports repo-specific findings, and can run clang-format
across eligible changed C/C++ files by default.

### decomp-diff.py — Diff & symbol overview

Overview mode lists all symbols in a translation unit with match status:

```sh
python tools/decomp-diff.py -u main/Speed/Indep/SourceLists/zAnim
python tools/decomp-diff.py -u main/Speed/Indep/SourceLists/zAnim -s nonmatching -t function

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbalatoni13/nfsmw](https://github.com/dbalatoni13/nfsmw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
