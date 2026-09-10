---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

C2 is an evolution of C (see http://c2lang.org). This repo is the C2 compiler (`c2c`), written in C2 itself, plus supporting tools (`c2tags`, `c2cat`, `c2loc`, `c2rename`, tester). Because `c2c` is written in C2, building it requires a two-stage bootstrap: a C-source bootstrap compiler builds the real `c2c`, which then rebuilds itself.

## Build & environment setup

Environment variables `C2_LIBDIR` and `C2_PLUGINDIR` must be set before building or running `c2c`. Always source `env.sh` first (it picks bootstrap paths vs. installed paths automatically):

```bash
. ./env.sh
make            # full bootstrap + build; output goes to output/
```

`make` does, in order: build `output/bootstrap/bootstrap` from `bootstrap/` (C sources) via `bootstrap/Makefile`, use it to compile `c2c` without plugins, install plugins (`./install_plugins.sh`), then rebuild `c2c` optimized with plugins enabled.

Useful Makefile targets:
- `make debug` / `make asan` / `make msan` / `make ubsan` / `make san` (asan+ubsan) — rebuild (`-B`) with `--debug`/sanitizer flags passed through as `C2FLAGS`.
- `make clean` — removes `output/` and `examples/output`.
- `make rebuild` — clean + all + test + trace_calls.
- `make errors` / `make warnings` — grep `error:`/`[-W...]` out of all `output/**/build.log` files.
- `make examples` — builds everything under `examples/` using the freshly built `c2c`.
- `make test-bootstrap` — builds a fresh `c2c_bootstrap` binary via `--bootstrap`.
- `make rebuild-bootstrap` — regenerates `bootstrap/bootstrap.c` and per-OS/arch patch files (darwin x86_64/arm64, freebsd, openbsd) from the real compiler; only needed when the bootstrap compiler itself must be updated.

Once built, run the compiler directly as `output/c2c/c2c` (or `output/c2c/c2c_fast`, the no-plugins intermediate build). `c2c` looks for a `recipe.txt` in the current directory describing build targets (see below); pass `--noplugins` to skip the plugin system, `--fast` for a faster/less-optimized build, `--quiet` to suppress build chatter.

## Tests

```bash
make                                # ensure c2c is built
output/tester/tester test           # run full test suite
output/tester/tester -q test        # quiet (Makefile `make test`)
output/tester/tester -v test        # verbose (Makefile `make testv`)
output/tester/tester test/vars      # run a subdirectory only
output/tester/tester test/vars/duplicate_function_arg.c2   # run a single test file
```

The tester (`tools/tester/`) recursively finds test files under `test/` and dispatches by extension:
- `.c2` — single-file compile test. Uses inline annotation comments:
  - `// @error{msg}` / `// @note{msg}` / `// @warning{msg}` on the offending line — expected diagnostics. Write a number after the argument if the message refers to a different line, eg: `// @warning{msg} +1` if the warning should occur on the next line.
  - `// @skip` — skip this test. Should be the first line.
  - `// @target{triplet}` — only run on a matching target.
- `.c2t` — tests generation of output files for a whole recipe/target. Requires `// @recipe bin/lib dynamic/static`, one or more `// @file{filename}`, and `// @expect{atleast|complete, filename}` blocks comparing generated output against an expectation file.
- `.c2a` — tests the parsed AST (unused-decl checking disabled). Requires exactly one `// @file{filename}` and one `// @expect{atleast|complete}`.

Test fixtures live under `test/<category>/` (e.g. `vars`, `types`, `expr`, `stmt`, `functions`, `c_generator`, `arch`, `parser`, `init`, `globals`, `template`, `attr`, `import`, `interface`, `module`, `naming`, `scope`, `builtins`, `literals`, `libs`, `auto_args`) — mirror this structure when adding new tests.

## Architecture

The compiler is organized as a classic multi-stage pipeline, one directory per stage; `recipe.txt` lists the exact file sets used to bootstrap-compile `c2c` itself and is a good map of dependencies between stages.

1. **`common/`** — shared infrastructure: `build_file`/`build_target` (parses `recipe.txt`), `source_mgr`, `diagnostics`, `console`, `string_pool`/`string_map`/`string_list`, `component` (a library/target unit), `file/` and `yaml/` subpackages.
2. **`ast_utils/`** — low-level AST support used everywhere: `string_pool`, `string_buffer`, `src_loc`, `attr`/`attr_table`, `context` (arena allocator for AST nodes), `color` (diagnostic coloring).
3. **`ast/`** — the AST node definitions themselves: declarations (`function_decl`, `var_decl`, `struct_type_decl`, `enum_type_decl`, `import_decl`, ...), types (`struct_type`, `array_type`, `pointer_type`, `function_type`, `qualtype`, ...), expressions (`call_expr`, `binary_operator`, `member_expr`, `init_list_expr`, ...), and statements (`if_stmt`, `for_stmt`, `switch_stmt`, ...), plus `module.c2` (a module = collection of decls) and `instantiator.c2`/`instance_table.c2` (template/generic instantiation).
4. **`parser/`** — hand-written recursive-descent parser: `c2_tokenizer` → `c2_parser` (+ `_expr`/`_stmt`/`_switch`/`_type` split files) produces the AST directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c2lang/c2compiler](https://github.com/c2lang/c2compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
