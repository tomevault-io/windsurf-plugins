---
trigger: always_on
description: This document describes the structure, conventions and workflows of this repository to help AI agents contribute effectively. The project is a matching decompilation of **Super Mario Sunshine** (GameCube, MWCC 1.2.5 C++ compiler, PowerPC Gekko target).
---

# Agent Guide for Super Mario Sunshine (SMS) Decompilation

This document describes the structure, conventions and workflows of this repository to help AI agents contribute effectively. The project is a matching decompilation of **Super Mario Sunshine** (GameCube, MWCC 1.2.5 C++ compiler, PowerPC Gekko target).

Important preliminary note: always prefer putting sentences in `.md` files on different lines to reduce the diff shown in git when rewording just one sentence — the lines still get collapsed into a single paragraph as per `.md` formatting rules.

## Project Goal

Reconstruct the original C++ source code such that compiling it with the original Metrowerks CodeWarrior compiler produces **byte-identical** object files to the original game binary (`mario.dol`). This is called a **matching** decomp.

### Mindset

The work is fundamentally about getting into the heads of the original developers using only the evidence available — the binary, the linker map, debug strings, and the compiler's behavior. Every assumption must be doubted and verified. Be skeptical of everything: naming guesses, code structure, control flow choices, even whether a function "looks right." Only commit to a decision when the evidence is overwhelming.

When progress stalls because a tough choice must be made and strong evidence is lacking, **leave the code nonmatching and move on**. New evidence often emerges later — from matching a neighboring function, finding a pattern in another TU, or discovering a debug string. Premature commitment to a wrong approach creates technical debt that's harder to undo than a TODO comment. The final goal is still always a 100% match, but trying to headbutt a particular function into matching 100% usually results in fakematches (see below) and technical debt. Case-by-case judgement should be used.

Truly hard judgment calls — ambiguous code structure, naming disputes, architectural decisions — should be deferred to humans.
Do that by leaving TODOs around places that feel especially fake and wrong rather than refusing to tackle complicated tasks.
The ultimate goal of the project is not just a matching binary, but **human-readable, modifiable source code**.

## Repository Layout

```
config/GMSJ01/
  config.yml          — decomp-toolkit project config
  symbols.txt         — layout of the binary, addresses, sizes, names etc for every symbol present
  splits.txt          — per-TU section address ranges
  build.sha1          — SHA1 hash of the target DOL

src/                  — decompiled C/C++ source files
include/              — headers (class declarations, inline functions)
orig/GMSJ01/          — original game disc image (not committed)
build/GMSJ01/         — build artifacts, compiled objects
  obj/                — target (original) object files extracted by dtk
  src/                — recompiled object files from our source
build/compilers/      — various retro compilers, downloaded during build
  GC/1.2.5            — the assembler, compiler and linker used for most of SMS
build/binutils/       — UNIX-style binutils that can be used for GC
build/tools           — various tools: dtk, sjiswrap

tools/
  configure.py        — (imported by configure.py at root)
  decompctx.py        — generates decomp.me context files
  project.py          — build system library

configure.py          — main build configuration; lists ALL objects and their matching status
docs/                 — documentation on reverse-engineering methodology
```

**MOST IMPORTANT NOTE OF ALL IMPORTANT NOTES**: autonomous agents are **strictly prohibited** from working on:
- MSL runtime, MetroTRK and THPPlayer,
- dolphin SDK,
- JSystem middleware.
Under the supervision of a human programmer, some degree of work might be acceptable on a narrow subset of these libraries known to be leak-free, but when working without a human programmer's supervision work on these libraries is strictly prohibited and such PRs will not be accepted. Work on game code instead.

## Build & Diff Workflow

Note that on windows, all tool names should be suffixed with `.exe`, while the following documentation uses the UNIX spelling (without `.exe`) for simplicity.

Ninja is used for most workflows. Simply running `ninja` in the root of the repository will build the project and report the overall matching progress.

To check changes against a baseline, `ninja baseline` can be used to generate a baseline report on the current state of the work tree, after which `ninja changes_all` can be used to show how the current changes to work tree influence matching progress, including a per-symbol report.
**Important**: for regression testing, you **must** use `ninja baseline` and `ninja changes_all`.
If the worktree already has changes and you want to test for regressions when compared to master, stash the changes, call `ninja baseline`, pop the stash and call `ninja changes_all`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doldecomp/sms](https://github.com/doldecomp/sms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
