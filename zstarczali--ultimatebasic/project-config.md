---
trigger: always_on
description: A custom BASIC-like language compiler targeting the Commodore 64 and Commodore 64 Ultimate. Produces `.prg` files runnable in VICE or on real hardware.
---

# NUltimate Basic

A custom BASIC-like language compiler targeting the Commodore 64 and Commodore 64 Ultimate. Produces `.prg` files runnable in VICE or on real hardware.

## Build & Run

```bash
cargo build --release
cargo test
ub build demo.ub -o demo.prg
ub build demo.ub --debug       # also generate .sym, .dbg and .vs debugger files
ub build demo.ub --asm         # also generate a readable 6502 codegen listing
ub build demo.ub --d64 disk.d64
ub build demo.ub --d64          # auto: demo.d64
ub build demo.ub --d64 disk.d64 --add music.prg --add loader.prg
```

## Project Structure

```
src/
  lib.rs               – public API: compile()
  main.rs              – CLI entry point
  compiler/
    mod.rs             – compile() + CompileOptions + CompileResult
    lexer.rs           – tokeniser  (Lexer → Vec<Token>)
    parser.rs          – AST builder (Parser → Vec<Stmt>)
    ast.rs             – Expr, Stmt, BinOp, ColorTarget, VarType enums
    codegen.rs         – 6502 code generator (Codegen)
examples/
  features.ub          – original feature demo
  new_features.ub      – arrays, word vars, sub params, string vars demo
  array2d_demo.ub      – multi-dimensional (2D) byte/word arrays, row-major indexing
  bitmap_demo.ub       – 320×200 bitmap, plot, circle, line
  block_demo.ub        – 80×50 block graphics, plot4, circle4, graphics on block
  joystick_demo.ub     – joystick reading, sprite movement
  mux_demo.ub          – raster sprite multiplexer (3 windows × 8 sprites)
  orbit_demo.ub        – 24-sprite orbit with pulsating radius
  plasma_demo.ub       – plasma-effect bitmap with raster bar animation
  sprite_data.ub       – sprdef shape data (included by other demos)
  sprite_mux_orbit.ub  – 24-sprite orbit with sprdef + precomputed positions
  sprite_orbit_demo.ub – 8 hardware sprites in circular orbit via sin/cos
  reu_bitmap_demo.ub   – REU stash/fetch with full-width (0-319) bitmap graphics
  cube_demo.ub         – tumbling 3D wireframe cube, double-buffered (graphics on double + flip)
  wide_x_demo.ub       – verifies plot/line/circle/rect reach X 256-319 (full 320 width)
  tenprint.ub          – 5 TENPRINT maze implementations with menu; demos lowercase charset mode
  text_scroll_demo.ub – hardware horizontal fine scroll text scroller
  fn_demo.ub          – text scroller rewritten with fn + typed string params
  function_demo.ub    – fn return value demo (square, add, max, clamp)
```

## Architecture

```
.ub source
  → Lexer::tokenize()  → Vec<Token>
  → Parser::parse()    → Vec<Stmt>
  → Codegen::compile() → Vec<u8>  (raw machine code)
  → mod.rs             → PRG = BASIC stub + machine code
```

### Two-Pass Compilation

1. **Pass 1** – every statement except `SubDef` / `FnDef` → main program body
2. `RTS` — end of main program  
3. **Pass 2** – only `SubDef` / `FnDef` statements → subroutines/functions appended after main

Sub bodies are never executed at startup. Forward references (`Call` to an
unknown name, `Goto` to an unknown label) are recorded and patched by
`patch_forward_refs()` at the end.

### Pre-Scan

Before either pass, `pre_scan()` walks the AST to:
- Allocate zero-page slots for every subroutine's and function's parameters
- Register arrays and assign their base addresses (`$C000+`)
- Allocate a 2-byte ZP return-value slot (`fn_ret_zp`) if any `fn` has a `: word` or `: float` return type

### Zero Page Layout

| Range | Purpose |
|---|---|
| `$00–$01` | CPU I/O port — never touch |
| `$02–$4F` | **Permanent**: variables, loop counters, for-limit/step, sub params (`perm_zp`) |
| `$50–$7F` | **Scratch**: expression evaluation, reset before each statement (`tmp_zp`) |
| `$FB` | RNG seed (LCG) |
| `$FC–$FD` | `fn_ret_zp` (2 bytes, if word/float fn present), else free |
| `$FE` | free |

`tmp_zp` is reset to `$50` at the start of every statement in `gen_stmts()`.
This prevents zero-page overflow into the KERNAL area (`$7A–$7B` = BASIC
current-line pointer).

### PRG Format

```
[01 08]            – load address header ($0801)
[0B 08][0A 00]     – BASIC line link ($080B), line number 10
[9E]               – SYS token
[32 30 36 31]      – "2061" (= $080D decimal)
[00][00 00]        – end of BASIC program
<machine code>     – loaded at $080D
```

### Array Storage

Arrays (`var a = array(N)`) are allocated from `$C000` upward — free RAM on
the C64 with no ROM overlay when no cartridge is present. All arrays are **zeroed
at program entry** (`emit_zero_arrays`, one shared loop over the whole array region),
because C64 RAM powers up with a garbage pattern.

---

## Language Reference

### Compile-time flags

```bash
ub build foo.ub --explicit    # require :type on every var / sub-param / fn-param
```

`--explicit` is a CLI flag (not a source-level keyword). When passed to `ub build`,
the parser refuses `var name = expr` and `sub foo(a, b)` / `fn foo(a, b)` without
explicit `: int|word|float|string` annotations. Arrays (`array(N)`, `array_word(N)`)
and `const NAME = value` are unaffected — their type is implicit in the declaration
form.

### Variables and Constants

```basic
var x = 10               # 8-bit integer (default)
var ptr: word = $0400    # 16-bit (two ZP bytes, lo/hi)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zstarczali/UltimateBasic](https://github.com/zstarczali/UltimateBasic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
