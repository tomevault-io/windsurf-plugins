---
trigger: always_on
description: Arcturus is a high-level interactive-fiction language with its own compiler,
---

# Arcturus

Arcturus is a high-level interactive-fiction language with its own compiler,
written in Python, that emits standard Z-machine version 5 story files. The
standard library is named Cosmos. The end goal is a complete, hackable IF
toolchain - compiler, editable library, a modern reference interpreter, and an
optional `arc_image` graphics path down to 8-bit machines - validated by porting
existing games to Arcturus: Hibernated 2 (text, the first large real-world game
and the maturity milestone; port complete) and The Curse of Rabenstein (from
DAAD, the graphics testbed). Ghosts of Blackwood Manor was dropped from the
plan once Hibernated 2 proved sufficiency (2026-07-04). Hibernated 2 releases
in its Arcturus build (Stefan's ruling, 2026-07-29). Writing Hibernated 3 stays
Stefan's own goal, pursued separately; this project stays focused on Arcturus
itself.

This file is the standing context for the project. Read it first, then the
three specifications under docs/, which are authoritative.

## Authoritative documents

- docs/00-roadmap.md: charter, locked decisions, milestones, the size
  strategy, and the graphics plan.
- docs/01-arcturus-handbook.md: the Arcturus Handbook, the one author-facing
  book: twenty-six chapters, one topic each, syntax first and library
  behavior after it, technical matter toward the back (the Designer's
  Manual arrangement). The two worked example games (chapters 25 and 26)
  are the conformance anchors. Cite it by chapter ("docs/01 chapter 14").

When code and a document disagree, the document wins: fix the code, or if the
document is wrong, propose the change and update the document in the same
commit.

## Locked decisions

- Target: conformant Z-machine version 5. The `--zversion 8` build flag targets
  version 8 for large modern-only releases; same codegen, only the version byte,
  file-length scale, and packed-address unit (4 to 8) differ.
- Smallest possible z-code is a primary objective, judged alongside
  correctness. Levers in order: whole-program dead-code elimination,
  abbreviation-based text compression, dense codegen (docs/00 section 5).
- The compiler is Python with zero runtime dependencies, so `arcturus` runs on
  a bare interpreter. Tests may use pytest as a dev-only dependency.
- Cosmos is the standard library, written in Arcturus itself and shipped as an
  editable template, not a black box. Overriding is ordinary handler
  resolution: most specific wins, `continue` defers to the next.
- No custom virtual machine, no transpile-to-Inform-6, no PunyInform runtime.
  The compiler emits z5 directly.
- Source extension .storyarc. Compiler binary `arcturus`, short form `arcc`.

## Repository layout

```
CLAUDE.md
docs/            00 roadmap + 01 handbook (authoritative); 03+ design records
arcturus/        the compiler package (lexer, parser, ast, sema, codegen, cli)
cosmos/          the Cosmos library in .prelude (english = language layer, actions,
                 parser, scope, dispatch, loop, core) plus the .granule features
actaea/          the reference interpreter (docs/06)
proteus/         the web interpreter, a vendored Z-machine-only Parchment fork
                 with arc_image (docs/09, PROVENANCE.md inside; node toolchain
                 for rebuilds, never for authors)
tools/           arcimg.py, proteus.py, the amalgamators, and other Python tools
examples/        brass-lantern.storyarc, cloak-of-darkness.storyarc
tests/           unit and golden tests
editors/vscode/  the syntax-highlighting extension
pyproject.toml
```

## Build, run, test

- Compile: `arcc examples/brass-lantern.storyarc -o build/brass-lantern.z5`
  (z5 by default; `--zversion 8` for version 8).
- arcimg image conversion needs Pillow ON THE MAC (`python3 -m pip install
  --user --break-system-packages pillow`); a Homebrew python upgrade wipes
  keg-installed packages, so if `import PIL` fails after an upgrade,
  reinstall to the user site. ACME (the C64 probe assembler) runs on the
  orb Debian machine from ~/FictionTools, its intended home; never install
  packages or tools in either environment without telling Stefan.
- Test: `pytest`.
- Verify a built story on a reference interpreter (Frotz or Bocfel); the same
  file must also run on Ceres for the 8-bit target.

## Coding standards

- Python 3.11 or later. Standard library only for the compiler runtime.
- Clear module boundaries: lexer, parser, AST, semantic analysis and the
  world-model IR, codegen, z5 story-file assembly, CLI.
- Plain ASCII punctuation in all generated code, comments, and documents. No
  em dashes; use commas, colons, semicolons, parentheses, or shorter
  sentences.
- Every milestone lands with tests and a green done-test before the next.

## Non-goals (do not build these here)

- Disk-image building or BuildTools integration. The project ends at a z5
  file; packaging is handled separately, later, elsewhere.
- A custom VM for running Arcturus games. They emit standard z5 and run on any
  conformant interpreter, so the compiler never ships a runtime. (The Actaea
  reference interpreter and arc_image graphics are their own later milestones,
  B10 to B12, built here but not during the language and library work.)

## Method


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ByteProject/Arcturus](https://github.com/ByteProject/Arcturus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
