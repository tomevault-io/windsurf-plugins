---
trigger: always_on
description: **mach** is an OCaml scripting runtime that automatically handles dependencies declared via `#require "..."` directives.
---

# CLAUDE.md

## Project Overview

**mach** is an OCaml scripting runtime that automatically handles dependencies declared via `#require "..."` directives.

## Build & Test

```bash
dune build # builds the mach executable
dune test # builds and runs tests, no need to run `dune build` first
dune promote # promote test outputs if we see it changed and it's expected
dune test test/test_build.t # can also run individual tests
```

## Testing

Tests are in `test/` as cram tests (`.t` files). Follow existing examples, only
add cram (`.t`) tests.

If you need to test something, create a new cram test `.t` file in `test/`.

### CLI Options

```bash
mach run script.ml [args...] # run script
mach run -v script.ml        # verbose mode (logs build commands)
mach run -vv script.ml       # very verbose mode
mach run -vvv script.ml      # very very verbose mode (shows build output)

mach build script.ml         # build without executing
mach build -w script.ml      # watch mode: rebuild on file changes
mach configure script.ml     # generate build configuration
mach pp script.ml            # preprocess source file to stdout (for merlin and build)
```

### Configuration

mach discovers configuration via:
1. `$MACH_HOME` environment variable (if set)
2. Walk up from cwd to find a `Mach` config file (like git finds `.git`)
3. Fall back to `$XDG_STATE_HOME/mach` (default: `~/.local/state/mach`)

### mach-lsp

LSP support for editors. Wraps `ocamllsp` with mach-aware merlin configuration:

```bash
mach-lsp              # starts ocamllsp with mach support
mach-lsp ocaml-merlin # merlin server mode (called by ocamllsp)
```

## Architecture

Split across multiple files:
- `bin/mach.ml` (~415 lines) - CLI entry point
- `bin/mach_lsp.ml` (~150 lines) - LSP/merlin support
- `lib/mach_std.ml` (~100 lines) - shared utilities (use `open! Mach_std` in all lib modules)
- `lib/mach_config.ml` (~115 lines) - configuration discovery, parsing, and toolchain detection
- `lib/mach_lib.ml` (~205 lines) - core implementation (configure, build, watch)
- `lib/mach_library.ml` (~65 lines) - mach library support (multi-module libraries)
- `lib/mach_module.ml` (~140 lines) - module parsing and require extraction
- `lib/mach_state.ml` (~235 lines) - dependency state caching
- `lib/mach_build.ml` (~250 lines) - build system implementation
- `lib/mach_ocaml_rules.ml` (~140 lines) - OCaml build rules
- `lib/mach_log.ml` (~10 lines) - logging utilities
- `lib/mach_error.ml` (~5 lines) - error handling
- `ppx_rule/` - PPX for build rule DSL

The library uses `(wrapped false)` so modules are accessed directly (e.g., `Mach_config`, `Mach_lib`).

### ppx_rule DSL

`ppx_rule` provides a DSL for expressing build rules. Used in `mach_ocaml_rules.ml`.

**Extensions:**
- `[%rule "..."]` - generates `Rule.rule` call
- `[%rule_dyndep "..."]` - generates `Rule.rule_dyndep` call (for dynamic dependencies)
- `[%cmd "..."]` - generates `Cmd.v` value

**Syntax in rule strings:**
- `>{target}` - target file (first appears in command)
- `>{target1|target2}` - multiple targets (first in command, rest silent)
- `<{dep}` - dependency file (first appears in command)
- `<{dep1|dep2}` - multiple deps (first in command, rest silent)
- `<{|dep}` - silent dep (not in command, just tracked)
- `<{deps...}` - dep list, uses `String.concat " " deps` in command
- `<{|deps...}` - silent dep list
- `%{cmd}` - Cmd.t fragment, merges its targets/deps
- `%{cmds...}` - Cmd.t list, uses `Cmd.concat cmds`

**Example:**
```ocaml
let ml = "foo.ml" in
let cmi = "foo.cmi" in
let cmx = "foo.cmx" in
[%rule "ocamlopt -c -o >{cmx|cmi} <{ml}"]
(* Generates: Rule.rule rules ~targets:[cmx; cmi] ~deps:[ml] ["ocamlopt -c -o %s %s" cmx ml] *)
```

### Mach_std Module

`lib/mach_std.ml` provides shared utilities used across the codebase. All library modules should have `open! Mach_std` at the top.

Key exports:
- `Filename.(/)` - path concatenation operator
- `Buffer.output_line` - write line with newline
- `Hashset` - simple hashset module (create, add, mem)
- `SS` - `Set.Make(String)` for string sets
- `SM` - `Map.Make(String)` for string maps
- `type 'a with_loc = { v: 'a; filename: string; line: int }` - value with source location
- `equal_without_loc` - compare `with_loc` values ignoring location (use with `List.equal`)
- `failwithf` - formatted failwith
- `command_exists` - check if a command exists in PATH
- `run_cmd`, `run_cmd_lines` - execute shell commands
- `mkdir_p`, `rm_rf`, `write_file` - file system utilities
- `type file_stat = { mtime: float; size: int }` - file stat info
- `file_stat`, `file_stat_exn` - get file stat
- `atomic_write_file` - write file atomically via temp file and rename

### Toolchain Detection

`Mach_config.toolchain` tracks:
- `ocaml_version` - from `ocamlopt -version`
- `ocamlfind_version` - from `ocamlfind query -format '%v' findlib` (None if not installed)
- `ocamlfind_libs` - map from package name to version (empty if ocamlfind not installed)

Libraries referenced via `#require "libname"` are validated at configure time against `ocamlfind_libs`. Errors include source location (file:line).

### Code Sections (lib/mach_lib.ml)

The code is organized with comment headers:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andreypopp/mach](https://github.com/andreypopp/mach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
