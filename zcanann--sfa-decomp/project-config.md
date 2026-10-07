---
trigger: always_on
description: Decompilation of Star Fox Adventures (GameCube): recover plausible original C that byte-matches the
---

# SFA-Decomp

Decompilation of Star Fox Adventures (GameCube): recover plausible original C that byte-matches the
retail binary. Main lib compiled with MWCC GC/2.0; audio/MSL with 1.2.5n.

## Goal & rules
- Recover the **plausible 2002 C** a Rare dev would have written. Inline `asm{}` is banned outside
  `src/dolphin/`. Inside SDK code, the only exception is paired-single `psq_l`/`psq_st` when MWCC has
  no intrinsic and a known-good donor or original binary proves the sequence. A clean-C 90% beats an asm 100%.
  The two retail assembly islands are handled as decomp projects handle authored asm, not as C to be
  matched: `src/main/zlb.s` is an assembled TU and `modelAnimBuildJointMatrices` in `src/main/render.c`
  is an MWCC function-level `asm` body, both byte-verified (`docs/foreign/`). Adding a third needs the
  same provenance case.
- Match % truth = `report.json` `fuzzy_match_percent`. Diff tools locate divergence; they don't certify it.
- **Fresh eyes.** This file is deliberately tiny and is **not** a catalogue of techniques or solved
  cases — that omission is on purpose. If a fix isn't here, that means nothing: read the target asm and
  derive it yourself. Every function is matchable; an unsolved one is a spelling not-yet-found, never
  an impossibility.

## The project itself is the playbook
The real knowledge lives in matched code, not in this file — that's why this file stays short. To
produce a particular asm shape, find code that already emits it and read the C behind it:
- **This repo's own matched functions** — grep `src/` for the construct, or disassemble a matched
  `.o` that has the shape you want (objdump command below) and open its source.
- **The MP4 reference decomp** (`reference_projects/marioparty4`, fully matched, same MWCC family) —
  a large known-good C↔asm corpus; read its source for how a given shape was written.
- **The GC/2.0 reference-asm corpus** (`tools/refcorpus/`, see `docs/refcorpus.md`) — ~42k funcs of
  SFA-adjacent C (MP4 + Diddy Kong Racing + Jet Force Gemini) recompiled with *our* compiler across
  the peephole×scheduling profiles. Search asm↔C both ways:
  `python3 tools/refcorpus/search_corpus.py --asm '<regex>' --show-c` or `--csrc '<c regex>'`.
A fix derived from a real matched example beats any written recipe, and the corpus only grows as the
project matches more.

## Build & verify
- Rebuild one unit + the report:
  `rm build/GSAE01/src/main/<path>.o && ninja build/GSAE01/src/main/<path>.o && ninja build/GSAE01/report.json`
- `ninja; echo EXIT=$?` — must be `EXIT=0` before any commit. (No `timeout` on this box: it is
  not installed, so a `timeout N ninja` gate returns 127 and never builds.)
- Paired-single disasm: `build/binutils/powerpc-eabi-objdump -M gekko -drz` (stock objdump mis-decodes PS as VSX).
- Tools are in `tools/` — start with `function_objdump.py <unit> <symbol>` (full target asm) and `ndiff.py`.

## Integration branch safety
- Retail target objs (`build/GSAE01/obj/...`) are READ-ONLY — never rebuild or delete them. Only the
  source objs (`build/GSAE01/src/...`) are yours to build.
- Normal work lands on the permanent branch `staging`, not directly on `main`. Fetch it from origin
  and rebase local unpushed work onto the fresh remote staging tip before every push; abort and
  re-derive conflicts instead of using `--theirs`.
- A maintainer or bot makes one normal merge commit from `staging` into `main` per UTC day, preserving
  per-change history while making the main Actions build run once for the batch. After publishing
  that merge, fast-forward `staging` to the new main merge commit before reopening it for work; do
  not create dated rollover branches. Reconcile any direct `main` commits into `staging` without
  dropping either side.
- Rebase + `ninja EXIT=0` before each commit; commit only when asked. One owner per `.c`.
- Edit SJIS-bearing files byte-wise (python rb/wb). Never `git stash` in a worktree — use `git checkout -- <file>`.
- **A function rename is NOT byte-neutral and can wreck a unit with a green build.** `config/GSAE01/symbols.txt`
  is build input (`config.split_deps`) — the splitter re-carves the retail objects from it, and objdiff pairs
  functions *and relocation targets* by name, so a source/symbols mismatch scores the function zero and shows a
  diff at every call site in every calling unit. Gate every rename with `python3 tools/pairing_check.py`
  (0 retail-only symbols) plus `unitfuzzy.py` on every unit `pairing_check.py --refs <name>` lists — not just the
  one you edited. See `docs/rename_safety.md`.

## Banned constructs (game code: `src/main/`, `src/track/`)
These are match-hacks, not plausible 2002 source. They were purged repo-wide (see
`docs/HACK_AUDIT.md`, tag `pre-hack-purge`) and MUST NOT re-enter:
- **Any `#pragma`** — per-function pragma sandwiches of every kind (peephole/scheduling/dont_inline/
  inline_max_size/opt_*/ppc_unroll_*/optimization_level/fp_contract/explicit_zero_data/force_active/
  exceptions). Inline pragmas are banned; pragmas may only be configured at the TU level via
  `configure.py` cflags.
- **`goto`** — write structured control flow.
- **`__declspec(section ...)`** and any section-forcing data placement.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zcanann/SFA-Decomp](https://github.com/zcanann/SFA-Decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
