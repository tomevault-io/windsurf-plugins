---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Rail Compiler

Self-hosting programming language. Compiler written in Rail, compiles itself to ARM64, x86_64, and Linux ARM64.

- **Compiler source**: `tools/compile.rail` (~4,690 lines, 335 functions)
- **Seed binary**: `rail_native` (729K ARM64) — checked into repo, self-compile produces byte-identical output (fixed point)
- **Native floats (v2.0)**: unboxed IEEE 754 doubles in ARM64 d-registers. No heap allocation. `fadd`/`fmul`/`fdiv`/`fcmp` directly. Float arrays, foreign float calls (`sin`/`cos`/`tanh`/`sqrt`), auto int→float promotion.
- **REPL**: `./rail_native run tools/repl.rail` — interactive, persistent definitions
- **HTTP server**: `stdlib/http_server.rail` + `tools/http_demo.rail` — compile handler binary, serve via `tools/http_server.py`
- **Error messages**: `file:line:col: error: message` — parse errors halt cleanly instead of segfaulting.
- **Runtime**: Zero C dependencies. GC is ARM64 assembly embedded in the compiler. Only needs `as` + `ld`.
- **GC**: Conservative mark-sweep garbage collector in ARM64 assembly. Scans stack frames, marks reachable tagged objects, sweeps into free list. Triggered when 512MB arena bump-alloc fails.
- **Allocator**: 512MB bump arena + GC free list + malloc fallback. 256MB thread stack. `arena_mark`/`arena_reset` still work.
- **Effect handlers**: `try body handler` — setjmp/longjmp non-local error recovery. Deep unwinding, nested handlers.
- **Type checker**: Forward inference pass emits warnings (not errors) for: head/tail on non-list, arithmetic on non-numeric, wrong arity, calling non-functions.
- **Package manager**: `import math` (bare imports), `rail get github.com/...`, `rail pkg` reads `rail.toml`.
- **Tests**: `./rail_native test` — 137 tests, should be 137/137. Count fluctuates only when concurrent sessions collide on `/tmp/rail_out` — rerun to confirm.
- **Checkpoints**: `stdlib/checkpoint.rail` — `save_checkpoint prefix weights adams step best_val` + `load_checkpoint` / in-place `load_model_into` / `load_adam_states_into`. Atomic via `<prefix>.committed` sentinel. `corpus_split text val_pct` for eval splits. `tools/train/lm_transformer.rail:run_segments` wires resume + periodic checkpoint into the training loop.
- **Performance**: Tail-recursive loops match C -O2 (5 instructions/iteration). Self-loop optimization, untagged register params, bottom-test with `subs`.
- **Targets**: macOS ARM64 (native), Linux ARM64 (Pi Zero), Linux x86_64 (Razer WSL)

### Key Commands

```bash
./rail_native test                    # run 137-test suite
./rail_native self                    # self-compile → /tmp/rail_self (must be byte-identical)
./rail_native run file.rail           # compile + execute
./rail_native file.rail               # compile only → /tmp/rail_out
./rail_native x86 file.rail           # compile to x86_64 Linux → /tmp/rail_x86.s
./rail_native linux file.rail         # cross-compile to Linux ARM64 → /tmp/rail_linux
./rail_native get <package>           # install package (stdlib name or github.com/user/pkg)
./rail_native pkg                     # install dependencies from rail.toml
```

### Rail Syntax Quick Reference

```rail
-- Comments start with --
add a b = a + b                       -- named function (BEFORE main)
main = let _ = print (show (add 3 4)) -- main returns int
  0                                       -- newline-based let
double x = let y = x * 2 in y            -- explicit 'in' also works

type Option = | Some x | None         -- ADT definition
getOrDefault opt = match opt           -- pattern match (NO 'with' keyword)
  | Some x -> x
  | None -> 0

fold add 0 [1,2,3,4,5]               -- fold (use named 2-arg functions, NOT nested lambdas)
map f list, filter f list             -- list ops
head xs, tail xs, length xs, reverse xs, cons x xs
range N                               -- [0..N-1]
\x -> x + 1                          -- single lambda OK
\a -> \b -> a + b                    -- nested lambdas work (flattened to multi-param)
write_file path content, read_file path
let _ = shell "command"
join sep list, split "c" str          -- split is per-character, NOT substring
str_split ", " str                    -- multi-char split
str_find "needle" "haystack"          -- returns index or -1
str_contains "needle" "haystack"      -- returns bool
str_replace "old" "new" str           -- replaces all occurrences
str_sub str start len                 -- substring extraction
read_line                             -- read line from stdin
show n                                -- int to string
int_to_float n                         -- tagged int → raw f64 bits (scvtf)
float_to_int x                         -- raw f64 → tagged int (fcvtzs, truncation)
x |> f                                -- pipe operator (f x)
error "msg", is_error x, err_msg x   -- error handling
arr_new size default, arr_get a i, arr_set a i v, arr_len a  -- mutable arrays
```

### Runtime Safety

- `head []` returns 0 (not segfault). `head` on non-list returns 0.
- `tail []` returns `[]`. `tail` on non-list returns `[]`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zemo-g/rail](https://github.com/zemo-g/rail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
