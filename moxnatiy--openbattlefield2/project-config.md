---
trigger: always_on
description: Licence: MIT (see `LICENSE`).
---

# OpenBattlefield2

Licence: MIT (see `LICENSE`).

Goal: a clean-room reimplementation of the Refractor 2 engine
(Battlefield 2, 2005) — an open engine that reads the user's own original
assets.

This is a **one-to-one port**, not a game "inspired by" the original. The
difference is fundamental and drives nearly every rule below: when we do
not know how something is done in the original, we write it down as debt
instead of filling it in with a guess.

## How the work goes

One pass is not "try something and see". It is four steps, and skipping
any of them is expensive:

1. **A measure.** First write down how we will know the task is done: a
   command, a number, a screenshot. "The enemy moves" is not a measure.
   "`--connect` prints a position update for another player's object more
   than once a second" is.
2. **A source.** The answer comes from the binary or from the game's
   data, not from memory (see "Sources of truth" and rule 12).
3. **Notes and code.** What was reversed goes into `docs/` and `src/`
   right away — **whole**, not just the side needed today (rule 3).
4. **Verification.** Run the measure. If it does not match, go back to
   the source rather than tweaking numbers.

A sign the pass went wrong: a third "let's try it this way" without a
single new look at the binary.

## Rules

### 1. Knowledge lives in files, not in the conversation

Never pull whole decompilations into the conversation. The order is:
search (a string, an xref, a table) → take apart **one** function with a
tool → write the notes into `docs/functions/<module>.md` — and do not go
back to the binary after that.

Write the notes **immediately**, while the binary is open. Every note
carries the address it came from; without one it cannot be checked.

### 2. Look whether the tool already exists

The list is below, under "Tools". This is not bureaucracy: the layout of
`readControlObjectState` was picked apart by hand from `objdump` output
even though `tools/linuxded/bitfields.py --blocks` prints it in one line
in 0.1 seconds. The tool was already there.

If the tool is missing — **make it first**, then work. One-off manual
work leaves nothing behind; a script is visible in git, can be re-run,
and shows where the answer came from.

### 3. Take a structure apart whole, not one field at a time

When a function, a format or a structure is open — write down **all** the
fields, **all** the mask bits, **all** the branches. Even the ones that
are not needed today.

This is arithmetic, not pedantry. One trip into a function costs the same
whether you take one field out of it or twenty. But when every next field
means going back, the work stretches over months — which is exactly what
happened to us: we went back into the controlled-object state three times
for one field each, and every time it cost the user a separate trip into
the game.

In practice that means:

* in the notes — a table of every field with sizes and the addresses they
  are read at;
* in the code — a structure with every field, even if part of it is
  unused (with a "not used yet" comment, not silently);
* fields whose purpose is unclear are written down too: size, offset, and
  an honest "purpose not established".

### 4. Do not reinvent what is already done

Before working on a format, check `docs/research/00-prior-art.md`. We use
the game's own data (`*_server.zip`, `*_client.zip`, `python/`,
`.con`/`.tweak`), the Project Dalian specifications (MIT) and BfMeshView.
Reverse engineering is only for what is not available openly.

`reference/` (not in git) holds third-party work for comparison. The
licences there differ: `breadflowerdos` has **none**,
`Refractor-2-BitStream-Emulator` is **GPL-3.0**; both are incompatible
with our MIT. It is a source of **hints**, not of code: everything taken
from there is verified against the binary or the game's data.

There is one exception: `reference/gameswf` is **public domain**. It is
the same library the original plays its Flash menu with
(docs/research/00-prior-art.md), so code may be taken from it directly.

### 5. Nothing is unpacked to disk

The game's archives are read in place through `obf2::FileSystem`, the same
way `fileManager` does it in Refractor 2. The `extract/` directory stays
empty.

### 6. No fudged numbers

Every constant in `src/` comes from one of three sources, and that source
is named in the comment next to it:

* the game's data (`.con`, `MemeFile`, a texture, localisation);
* the binary (an address in `BF2.exe`, `BF2_r.exe` or the Linux server);
* a direct measurement from a frame dump of the original
  (`Ctrl+Shift+D`).

Picking a number so that it "looks about right" is forbidden, even when
the result matches visually. With no source, leave it as it is, write
**"not measured"**, and say so out loud.

An indirect conclusion is fudging too. Example: we took a triple of
numbers in the controlled-object state for a position because it matched
the spawn camera. It is in fact the compression origin, and the soldier
was thrown a metre into the air on every packet.

### 6a. Every comment names the address it came from

Any constant, any field, any layout, any rule of behaviour in the code is
accompanied by **the address in the binary** (or the path to the data
file) it came from. Not "from the client", but `BF2.exe, 0x62d4e0`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Moxnatiy/OpenBattlefield2](https://github.com/Moxnatiy/OpenBattlefield2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
