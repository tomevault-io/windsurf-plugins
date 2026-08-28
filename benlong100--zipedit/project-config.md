---
trigger: always_on
description: An 80-column, full-screen **Markdown** editor for the Enhanced Apple //e
---

# a2-editor

An 80-column, full-screen **Markdown** editor for the Enhanced Apple //e
(128K, ProDOS 8), **written in** 6502 assembly in Merlin syntax. Files are
drafted on the //e and moved back to a Mac for publishing.

Note the distinction: this is a text editor *implemented in* assembly, not an
editor *for* assembly source. It is aimed at prose. Hard wrap on entry, no soft
wrap, and Markdown-aware emphasis shortcuts.

See `docs/design.md` for the memory map, buffer design, and keymap.

A splash screen shows at startup and waits for a key, and the editor then opens
on an EMPTY document — the sample lives on the disk as `SAMPLE.MD`, which is
where the suite gets its text. Anything driving the editor has to get past the
splash first — `reboot` in the suite
does that.

## Toolchain

Source lives here on the Mac and is cross-assembled; the //e never sees a
source file we didn't generate.

- **Merlin32** (`tools/merlin32`) — primary assembler. Built from
  `apple2accumulator/merlin32`.
- **AppleCommander 13.2** (`tools/ac`) — builds ProDOS disk images. Native
  arm64 binary, so no JRE is required.
- **Virtual ][ 11.4** — emulator, driven through AppleScript by `tools/vii.sh`.
- **Merlin 8 v2.48 (DOS 3.3)** — `vendor/`, the user's original assembler. Kept
  as the dialect reference, not used in the build.

`tools/bootstrap.sh` reconstructs all of it on a fresh clone.

## Commands

```
make            assemble $(SRC) with Merlin32
make disk       build a bootable ProDOS 8 image at build/ZIPEDIT.po
make run        build, boot in Virtual ][, print the screen
make test       run the regression suite
make pull       eject, then extract Markdown from the image into notes/
make push       convert notes/*.md back onto the image as ProDOS TXT
make eject      flush the mounted image to disk
make clean
```

`SRC` defaults to `src/edit.S`, the editor itself. The spikes are still
buildable and worth keeping: `make SRC=src/charset.S run` re-runs the character
generator probe, `src/hello.S` is the minimal toolchain check.

## Conventions

- **Merlin dialect.** Merlin32 is Merlin 16+ flavoured and is not identical to
  the Merlin 8 on the //e. Stay in the common subset: plain 6502, and the
  directives `ORG EQU DFB DW DS ASC HEX LUP MAC EOM`. Treat Merlin 8 as ground
  truth if the two ever disagree.
- **Case and layout.** Lowercase mnemonics, labels in column 1, opcodes at
  column 14, operands at column 20, comments at column 40 — matching the
  Merlin house style in `src/hello.S`.
- **Local labels** are `:name`, scoped to the enclosing global label.
- **Never hand-edit the help screen hex.** `src/help.S` holds 42 rows of raw
  screen codes and they are generated: edit the layout in `tools/genhelp.py`
  and re-run it. It enforces the column stops, so a description that would
  overrun the border fails there instead of shipping mangled.
- **Line breaks come in three kinds.** `HARDCR` is a return the writer typed,
  `SOFTCR` a wrap that replaced a space, `SOFTWD` a wrap inside a long word.
  Only `HARDCR` reaches the file. Test for a line end with `cmp #TEXTLO / bcc`,
  never `cmp #$8d` — a missed one is a break that some subsystem cannot see.
- **Never hard-code a screen column.** The layout's columns -- the wrap
  margin, the status row's fields, the prompt's hint, the help box's edge --
  live in the record in `src/geom.S`, filled at startup from a table. An
  equate compiled into the code that reads it is an 80-column assumption, and
  there is a 40-column machine coming. Rows are different: 24 either way, so
  `CHEATROW`, `STATROW` and `TEXTROWS` stay equates. `SCRW` survives only as
  the size of the widest line we allocate.
- **High ASCII.** Anything destined for the screen or for a text file has bit 7
  set. `asc "..."` (double quotes) sets it; single quotes do not.

## Keymap

Arrows move by character and line. OA-up/OA-down page, OA-`<`/OA-`>` jump to
the start and end of the document. Ctrl-A/Ctrl-E line ends, Ctrl-B bold (`**`),
Ctrl-I italic (`*`), Ctrl-D delete forward. OA-R reflow, OA-S save, OA-O open,
OA-C/OA-X/OA-V copy, cut and paste a line. OA-F find, OA-G find again, OA-L go
to line, OA-W word count, OA-Delete deletes back to the start of the word. OA-N starts a new document and OA-Q quits, both asking first if the document
is modified. OA-S saves to the document's own file and only prompts when it has
no name yet; OA-A is save as, which always prompts and adopts the new name.
Anything that replaces the document -- OA-N, OA-Q, OA-O -- must go through
ASKUNSAVED first. OA-? (or OA-H) opens the keyboard help,
which is two pages -- a key turns to page two, another leaves. OA-/ toggles the
one-line cheat sheet, OA-Q quits. **OA-`'` types a backtick** -- no Apple II
keyboard has a grave-accent key, so this is the only way in; three of them
make a fence. The ][+ spells it `Esc '`, and draws it as an apostrophe because
that character generator has no glyph for it. `$89` is both Tab and Ctrl-I and dispatches on
position -- see `docs/design.md`.

## Testing

`tests/run.sh` boots the built image in Virtual ][ and asserts against both the
emulated screen and emulated RAM. Prefer RAM assertions — `vii.sh dump <addr>
<len> <bank>` reads the auxiliary bank, so tests can check the text buffer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benlong100/zipedit](https://github.com/benlong100/zipedit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
