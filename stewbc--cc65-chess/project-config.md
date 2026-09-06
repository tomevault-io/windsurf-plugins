---
trigger: always_on
description: A chess program in C, originally for ~1 MHz 6502 machines — c64, apple2, atmos, plus4,
---

# Working on cc65 Chess

A chess program in C, originally for ~1 MHz 6502 machines — c64, apple2, atmos, plus4,
atari. Faster 6502s came later: cx16 and rp6502, at up to 8 MHz. Then ports that cc65
does not cover — spectrum, mac68k — plus a terminal build used for development. Each
port with whichever compiler that machine needs.

`doc/engine.md` explains how the engine works and is the right thing to read first. This
file is the short list of constraints that are easy to violate by accident.

`doc/rework-log.md` is the working journal, and its last section is the closed search and
evaluation portfolios. Read that before proposing a search technique or an evaluation term.
B3+B4 shipped (−17.2% per node on a C64). The skill budgets are 400 / 1,200 / 18,000 /
65,000. Do not reopen a rejected candidate by renaming it.

## Hard constraints

**`plat.h` and the 0–63 tile numbering are frozen.** The 0x88 board representation stays
inside the engine and converts at the boundary. This is what lets the platform files that
cannot be built here stay untouched.

**There is one addition to `plat.h` since the freeze, and the bar it had to clear is the
point.** `plat_GetSeed()` returns a byte of entropy for the opening randomiser. It went in
because every machine has a free-running counter, cc65's own `asminc` names all of them, and
the implementation is three lines a port that reads a register and cannot fail visibly — a
wrong address gives repeating openings, which is what the engine did anyway. Anything asking
to be added here should be held to that: no writes to hardware on a port that cannot be run,
and a failure mode nobody has to debug. Note there are **eleven** platform files, not seven —
`c64.chr` is a separate port, `spectrum` and `mac68k` use other compilers, and `term` is
the one that must return a constant.

**The frozen interface is wider than `plat.h`.** Every port also reads four globals
directly, and this was discovered the hard way:

- `gChessBoard[y][x]` — pieces, for drawing
- the attacker **count** for a tile and side — no port ever reads the attacker list
- `gTile[0]`, `gTile[1]`, `gPiece[1]`, `gColor[0]` — the move log line, valid only inside a
  `undo_FindUndoLine` walk; see the log contract below

There is a fifth, and it is the shape a new one has to be: `gGotoTile` is *written* by a port
to jump the cursor from a pointing device, and it is behind `PLAT_CURSOR_JUMP` so that the ten
machines with no mouse do not compile it at all.

**`cx16` builds here now but is still not *run* here.** The build failure that predated the
engine work was cc65 renaming the software stack pointer: `platCX16.c` used `(sp)` in inline
assembly where current cc65 wants `(c_sp)`. It compiles and links clean, repetition detection
included. Running it still needs the Windows machine, so the rest of the rule stands — do not
delete it, do not knowingly break it, do not edit its platform files speculatively.

**Two targets have video memory the linker cannot see, and both now have a config that
says so.** The Apple II's HGR page 1 at `$2000` and the Atari's GR.8 framebuffer at a
hard-coded `$9100` in `hiresAtari.s`. A clean `ld65` run proves nothing about either: the
Atari linked "inside its budget" for months while drawing BSS onto the top of the screen.
Both cfgs now cap the program below the framebuffer, so an overrun is a link error.

**`rp6502` is the one target with no framebuffer to collide with at all** — its video memory
is XRAM on the VGA co-processor, off the 6502 bus, reached through `RIA.addr0`/`RIA.rw0`. But
a clean `ld65` still does not prove everything there either: stock `rp6502.cfg` runs its RAM
area to `$F6FF` and `crt0.s` then starts the 2K C stack at `$F700` growing *down* into it, so
the real ceiling is `$EF00` and the last 2K of the linker's own area is not free. With 26K
spare that is a note, not a risk.

**Every target is built at the same optimisation setting, and that setting is `optsize`.**
Uniformity is the point - a port that is built differently is a port that behaves
differently - so `optspeed` is not an option for one target while another cannot take it.
Since the endgame tables it is also the only setting that fits:

| | optsize | optspeed |
|---|---|---|
| atari | 874 free below the framebuffer | **does not link — 1942 bytes over** |
| apple2 | 660 free in MAIN, 2024 in BSS | **does not link — 1031 bytes over** |

**The Atari's headroom does not shrink smoothly, and the number above hides a cliff.** `DLIST`
is page-aligned inside `MAIN`, so code growth is absorbed by the padding in front of it until
the padding runs out and the display list jumps a page — 256 bytes, gone at once. That has
happened three times: the opening table crossed it, check evasions crossed it again, and the
black reply table crossed it a third time, which is why the Atari lost 256 bytes each time
while the Apple II lost the 302, 132 and 253 the code actually costs. It happened a fourth
time and was **taken back** — see the port-tax rule below; the Atari is the reason that rule
exists.

**There are 42 bytes of padding left.** Check `DATA`'s end against `DLIST`'s start in an
`ld65 -m` map rather than reading the free-space number on its own; the two tell different

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StewBC/cc65-Chess](https://github.com/StewBC/cc65-Chess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
