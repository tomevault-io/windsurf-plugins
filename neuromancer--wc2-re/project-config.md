---
trigger: always_on
description: You are reconstructing the source of **Wing Commander** as shipped in *Wing Commander: The
---


## Project Context

You are reconstructing the source of **Wing Commander** as shipped in *Wing Commander: The
Kilrathi Saga* (1996), the Win32 port of the 1990 DOS original. Confirmed by the registry key
the binary reads: `Software\Origin Systems\WC: Kilrathi Saga`.

- Original working title: **WINGLEADER**, © 1989,1990 Chris Roberts (Origin Systems).
- Original codebase: **C** for the game core, **C++** for the `ix` audio library.
- Compiler: **Microsoft Visual C++ 4.20**, static *debug* multithreaded CRT (LIBCMTD).
- The shipped executable is a debug build: live `assert()`s, the MSVC debug heap, and a
  `\\.\MONODEBG.VXD` developer channel are all present.
- ~1,450 developer-written functions; all are named in the Ghidra database, but only ~437 of
  those names are evidence-backed. The other ~1,013 are `<Verb><Object>Fn<addr>` operational
  labels that describe mechanism, not purpose. **Do not trust an operational label as a
  statement of intent.**

IMPORTANT: The assembly output and the extracted strings are the *only* source of truth.
Decompiled code is a useful hint but is NOT authoritative.

The Ghidra-exported disassembly is in `code-full`. The reimplementation lives in `src`
(game core, C) and `src/ix` (audio library, C++).

---

## Absolute Rules (Do Not Violate)

- DO NOT remove or modify a function whose comment header carries a WC2 address
  (`/* Function start: 0x... */`). Those are verified against the retail image.
- DO remove `WC2_UNMAPPED` functions and WC1-address globals once nothing that
  survives can reach them: this branch is a WC2 reconstruction, and inherited WC1
  code that WC2 never runs is noise. Removal must be closure-aware -- compute what
  is reachable from the mapped functions and the file-scope tables, iterate to a
  fixpoint, and confirm `make verify` still passes. Never remove a `WC2_UNMAPPED`
  function that a mapped function still calls: that call is either a defect to fix
  or evidence the function is really WC2 code awaiting an address.
- DO NOT remove code that is still reachable.
- DO NOT change calling conventions. The game core is `__cdecl`; a handful of `ix` functions
  are `__thiscall` — leave those implicit, never spell `__thiscall` out.
- DO NOT add:
  - inline assembly, EXCEPT where there is strong evidence the original was hand-written
    assembly (see docs/PATTERNS.md)
  - dummy variables
  - helper functions or wrappers that do not exist in the original binary
  - vtable fields or manual vtable handling
  - unions or substructures
  - `try`/`catch`/`__finally` (see the SEH note below)
- DO NOT show the final code once you finish.
- If you are out of ideas, stop. Do not break any rules.
- In C++ (`src/ix`), do not use `this->`; use the class field name directly.
- When a funciton is implemented, it should have a good name. No completely generic names are acceptable, so you must investigate globals and such to understand how to name them.
- When a new .c file is added, it should have a good name. No leafsX.c are accepted.
- Do NOT put a function in only one line `unsigned short f(void) { return 0; }`. The
  reconstruction is read side by side with the disassembly, so one source line per statement
  is what makes the two comparable. `make sort` fails if any remain;
  `bin/expandOneLiners.py` rewrites them.
- DO NOT uses aliases, rename instead. In particular, do not use aliases for globals (#define g_X_Y ((void *)DAT_Y)). Instead rename all of them with a proper type.
- Do NOT manually replicate thunk functions or other compiler generated glue code (e.g. GetFixedOneMillionThunkAlt(void) { __asm { jmp GetFixedOneMillionAlt }). These need to be produced by the compiler automatically, not forced.

### WC1-specific rules that differ from sibling projects

- **`.c` files are correct here.** The game core was C — the leaked WINGLEADER main module is
  a `.c` file including `<game.h>` and `<dos.h>`. Only `src/ix/*` is C++. Do not "upgrade"
  game-core files to C++ or to classes.
- **Never add C++ exception handling.** The image contains no `__CxxFrameHandler` and no RTTI
  type descriptors (`.?AV`), so `/GX` was off. The SEH that does exist is C `__try`-style
  `_except_handler3` scope tables emitted by the compiler; do not write it by hand.
- **Do not rely on identical string literals being merged.** `/Gf` was NOT used: two
  byte-identical `"DIBsetWholePalette   SetEntries"` literals exist at `0x0046b6e0` and
  `0x0046b71c`. Write each literal out at its own use site.
- **Expect 16-bit types.** The core was ported from 16-bit DOS C, so `int` was 16 bits
  originally. Most game state is `short`, and 814 functions carry a measurable 16-bit-operand
  density. If your build emits 32-bit operations where the original used 16-bit, the type is
  wrong — use `short`, not `int`.
- Global variables may be renamed from `DAT_x` to `g_<hungarian><Name>_<address>`; the
  address MUST be preserved in the name. Functions must NOT carry the address in their name.
- **Name every function you implement.** An operational label (`DoLocalFn5450`,
  `HelperOf430FC0C`, `ReturnConst0v5`) is acceptable only while a function is an unwritten
  stub. Once written it takes the developer's own name if the binary states one

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neuromancer/wc2-re](https://github.com/neuromancer/wc2-re) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
