---
trigger: always_on
description: AI agent guidance for this codebase. For project docs, build instructions, and architecture, see [README.md](README.md).
---

# AGENTS.md

AI agent guidance for this codebase. For project docs, build instructions, and architecture, see [README.md](README.md).

## Project Overview

ACE is a BASIC compiler for Amiga. It compiles BASIC source code (.b files) into native Amiga executables by generating Motorola 68000 assembly. The compiler is written in K&R C.

**Build Pipeline:**
```
Source (.b) → Preprocess (yap) → Compile (ace) → Assemble (vasmm68k_mot) → Link (vlink) → Executable
```

The `bas` wrapper script in `bin/` orchestrates this full pipeline.

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `src/ace/c/` | Compiler source (lexer, parser, code generator) |
| `src/lib/c/` | Runtime library C sources |
| `src/lib/asm/` | Runtime library assembly sources |
| `src/make/` | **Build from here** - Makefiles and build scripts |
| `bin/` | Compiler binaries and bas wrapper script |
| `lib/` | Built libraries (db.lib, startup.lib) |
| `bmaps/` | Binary maps for Amiga shared libraries |
| `include/` | Amiga system headers and submodule headers |
| `submods/` | Submodules (reusable BASIC libraries) |
| `verify/tests/` | Test suite (cases/, expected/, results/) |
| `verify/scripts/otherthenamiga/` | Emulator and Amiga system files |
| `docs/` | Documentation (ref.txt for language reference, quickref.txt) |
| `examples/` | Example programs by category |
| `IDE/CubicIDE-ACE/` | CubicIDE integration (syntax highlighting, autocase, quickinfo) |

## Workflow - CRITICAL

**Work in small, incremental steps. Only proceed when the previous step is verified working.**

1. Think test-driven: create a test first that specifies the implementation
2. Make ONE change at a time
3. Verify it works (build, test, or run)
4. Only then proceed to the next change
5. Every plan/spec is to be implemented in a separate git branch

Why: The Amiga environment is fragile - path handling, toolchain differences, and AmigaOS quirks mean changes interact unexpectedly. Small steps make debugging and rollback feasible.

### Verification After Each Change

- **Build changes**: Run a build, check executable exists
- **Script changes**: Execute the script, check output
- **Test changes**: Run the test suite
- **Compiler/library changes**: Build and run relevant tests (must run on emulator)

Note: Compiler/runtime rebuild is only necessary when its source files were changed.

## Amiga Emulator Testing

### Setup

- **Emulator app**: `verify/scripts/otherthenamiga/FS-UAE.app`
- **Config file**: `verify/scripts/otherthenamiga/ace-verify.fs-uae`
- **Amiga system**: `verify/scripts/otherthenamiga/aos3`
- **Startup script**: `verify/scripts/otherthenamiga/call-on-ustartup` - edit this to run commands on boot (called from `aos3/S/user-startup`)

### Running the Emulator

```bash
# Start the emulator
open verify/scripts/otherthenamiga/FS-UAE.app --args verify/scripts/otherthenamiga/ace-verify.fs-uae
```

### Testing Workflow

1. Edit `verify/scripts/otherthenamiga/call-on-ustartup` to run your test commands
2. Write output/logs to `ace:` (maps to project root on host)
3. Start/restart emulator
4. Periodically check for result files (every 30 secs)
5. Runs take 5-10 min when recompiling, <1 min otherwise

**IMPORTANT: The emulator MUST be restarted after every change to `call-on-ustartup`.** The startup script is only read once at boot time.

### Example call-on-ustartup for testing

```
; In verify/scripts/otherthenamiga/call-on-ustartup
cd ace:submods/mui
bas test_minimal >ace:test-output.txt
test_minimal >>ace:test-output.txt
```

It may be necessary to call the commands of bas individually, i.e. to get the assembler source code .s.

A module must be compiled using `-m` switch: `bas -m mymod`.

### Using `-E` flag for compiler errors

`bas -E myfile` makes ace write compiler errors to `ace.err` in the current directory. **IMPORTANT: `ace.err` is overwritten on each `bas -E` call.** When compiling multiple files, save or append the errors after each compilation:

```
; Compile module, save errors
bas -mEO mymod >ace:build-output.txt
type ace.err >>ace:build-output.txt

; Compile test, save errors separately
bas -E test_foo >>ace:build-output.txt
type ace.err >>ace:build-output.txt
```

Without this, only the last file's errors will remain in `ace.err`.

## ACE BASIC Syntax

Full reference: `docs/ref.txt`

Key points:
- Comments: `REM` or `'`
- Variables: `DIM x AS INTEGER`, `DIM s AS STRING`
- Strings end with `$`: `name$`, `DIM text$ AS STRING`
- Arrays: `DIM arr(10) AS SINGLE`
- Structures: `DECLARE STRUCT mystruct`, access with `->`
- Library calls: `LIBRARY "library.library"`, `DECLARE FUNCTION`
- Labels for GOTO/GOSUB: `label:`
- Subprograms: `SUB name ... END SUB`, `FUNCTION name ... END FUNCTION`
- External modules: `EXTERNAL modulename`
- calling SUBs needs parenthesis
- don't use END inside IF blocks, use STOP.

## Pitfalls for AI Agents

### AmigaDOS Specifics

- **No stderr redirect**: `2>&1` doesn't work on AmigaDOS
- **No .b extension**: Call `bas myprog` not `bas myprog.b` (on emulator)
- **Debug build phases**: Split bas phases (ace, vasm, vlink) and redirect each to separate files to see what's happening

### Amiga Path Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mdbergmann/ACEBasic](https://github.com/mdbergmann/ACEBasic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
