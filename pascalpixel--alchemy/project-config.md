---
trigger: always_on
description: enables the reconstructed `-mgs2` lowering in agscc. Code the ROM copies into
---

# Alchemy

Alchemy is a decompilation of Golden Sun: **The Broken Seal (TBS)** ☀️ and
**The Lost Age (TLA)** ⚓️. It rebuilds each game byte for byte from readable C,
assembly and editable assets. Japanese releases are the source editions;
localizations are measured differences. Build IDs are `tbs` and `tla`.

This file is the only working guide. `README.md` is for fans. Read this file
once; do not copy it into prompts or notes.

## Our model: pret's pokeemerald

We work the way pret's pokeemerald did from 2015 to 2021, when it reached 100%
(a local checkout lives at `~/Developer/pret/pokeemerald`). When in doubt about
method, tooling, what to commit or how to measure, do what pokeemerald did.

The one deliberate difference is presentation: our tree looks like the project
Camelot most plausibly had on disk in 2001, not like pret's layout.

| pokeemerald | Alchemy |
| --- | --- |
| `src/`, `include/`, `asm/`, `data/`, `graphics/`, `sound/` | `games/<GAME>/SRC`, `INCLUDE`, `SOUND`, `TEXT`, assets beside their module; uppercase 8.3-style names such as `BATTLE/EFFECT/PARTICLE.C`, `FIELD_EVENT.H`, `CHAR_ISAAC.PNG` |
| snake_case files, CamelCase functions | uppercase files; `Subsystem_VerbObject` functions; short `pos`, `cnt`, `tbl`, `buf`, `work` locals; C89 |
| English map names | romaji place prefixes plus one area word from the Japanese ROM: `RUNPA_DOU`, `HAIDIA_MURA` |
| scaffolding in the same tree | our scaffolding lives apart in lowercase `recon/tbs` and `recon/tla` and shrinks to nothing at 100% |

## How we work

1. **Match first, polish later.** A function counts when its C compiles to the
   exact bytes. Names, types and comments improve over time, as pret's did; a
   placeholder name is fine until evidence gives a better one.
2. **Never throw work away.** A function that does not match yet keeps its C
   beside the assembly the build still links, pret's `NONMATCHING` pattern:
   the C lives in the module under `#ifdef NONMATCHING` (or in its
   `recon/<game>` draft) with its score and remaining difference named. It
   counts when it matches. A readable rewrite of a matching function is kept
   the same way until it matches too. Every attempt ends in a commit.
3. **Fake matches are allowed and tagged.** C that matches only through an
   odd construct (a reordered statement, a temporary that means nothing, a
   `register` hint) counts, carries a `/* FAKEMATCH */` comment saying what is
   odd, and is cleaned up later. What never counts: patched compiler output,
   bytes copied from the ROM into source, and changing the test or the
   measurement.
4. **Largest functions first.** Rank the unmatched functions by size, take the
   largest with the best evidence, and reuse what exact neighbours, callees and
   shared headers already prove. After three attempts without a new idea,
   commit the best draft and take the next one.
5. **Stop researching when it stops paying.** Work that cannot raise the
   percentage (compressor tails, packing, provenance archaeology) is timeboxed
   and never blocks a commit.

## Compiler and build

Game code uses the approved agscc bundle (GCC 2.96 based) through
`tools/alchemy/src/compiler/routing.rs`. As in pokeemerald, a whole source file
may use its own flags when the original evidently differed (pret builds its
library and flash files that way); record the reason beside the route. TLA
enables the reconstructed `-mgs2` lowering in agscc. Code the ROM copies into
RAM and runs in ARM state uses `-marm -mno-apcs-frame` (Pascal, 2026-09-23).
Compiler source changes, new binaries and new digests need Pascal's approval.

The build must always reproduce both English ROMs byte for byte:

```sh
./alchemy build full                 # TBS English
./alchemy build full --target tla-en # TLA English
make verify                          # the commit gate
```

Other editions compile but do not rebuild yet.

## Assets and compression

Assets are editable files the build converts, as in pokeemerald: PNG graphics,
JSON tables, MIDI sequences, WAV samples and PO text. Compressed data is
regenerated from them by our encoders. Where the general encoder does not
reproduce Camelot's stream, a per-file option (a window, a search limit, a
read-ahead) is recorded beside that file, as pret records `-num_tiles`; that
is a build setting, not cheating. Stored raw streams from the frozen
compression answers (TBS, commit `d08ee3a2`) and the six stored TLA streams
(Pascal, 2026-09-23) remain until an option or the encoder replaces them.

Game assets are tracked in the repository exactly as pokeemerald tracks them,
in the same shape: one indexed PNG per asset with its real palette (a
character, a portrait, a tileset), one identified BIN per tilemap or table,
and WAV, MIDI, JSON and PO. Never move a tracked asset out of the tree or
replace it with ROM extraction. What pret would not commit stays private: the
ROMs, the cartridge logo, and raw dumps. Today's giant grey sheets
(`CHAR_COMMON.PNG`, `TILE_BANK.PNG`), whole-area BIN bundles and the
unidentified `DATA.BIN` are dumps, restored from your own ROM
(`recon/<game>/private-inputs.json`, `--extract-sources`) until each is split
into pret-shaped assets and moved into its module.

## What may be committed


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PascalPixel/alchemy](https://github.com/PascalPixel/alchemy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
