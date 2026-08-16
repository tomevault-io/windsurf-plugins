---
trigger: always_on
description: Read `README.md` first for repo layout and status. This file is
---

# Agent instructions for rocm-gfx803

Read `README.md` first for repo layout and status. This file is
process/judgment guidance for working in this repo specifically -- the
parts that aren't obvious from the code or the patch headers alone.

## Standing philosophy

- **Fix at the source. No workarounds.** A gfx803 bug gets fixed where it
  actually lives -- the broken Tensile logic, the broken MIOpen solver, the
  broken MIGraphX pass -- not papered over with a retry loop, a narrower
  input-shape gate that avoids triggering it, a fallback to a slower-but-
  working path chosen defensively without knowing if the fast path is
  actually broken, or a try/catch that swallows the failure. If you can't
  fix it at the source (upstream code you don't control, or a genuine
  hardware limitation), say so explicitly and stop -- don't ship a
  workaround dressed up as a fix. `wgm-miscompute.sh` is the model:
  root-caused down to the exact broken instruction sequence, fixed by
  correcting the actual parameter Tensile computes wrong, not by avoiding
  the shapes that expose it.
- **Correctness costs performance sometimes -- that's fine, up to a point.**
  5-10% throughput lost to a correctness fix is an acceptable, expected
  trade, not something to negotiate around. Past that, flag it explicitly
  and ask rather than silently accepting a large regression or silently
  picking a workaround to avoid it. Never trade correctness for speed
  without saying so out loud first.
- **Comments say WHY, never WHAT.** Code should be legible enough that the
  WHAT doesn't need restating in prose next to it -- a comment that
  describes what the next three lines do is dead weight the moment the
  code changes and stops matching it. Write a comment only when there's a
  non-obvious reason behind the code: why this bound, why this order, why
  this workaround-shaped thing is actually correct here, what broke last
  time someone tried the obvious approach. This repo's patch headers are
  the reference model -- WHY (how the bug was found, what it looks like)
  before WHAT (the actual diff).
- **Comments are not commit history.** Don't write "changed X to Y",
  "added this for the Z fix", "removed unused W" -- that's what `git log`
  and `git blame` are for, and it rots the moment the described change is
  no longer the most recent one. A comment should make sense read cold, by
  someone with no idea what the last edit was.
- **Never comment on absence.** Don't write a comment explaining that
  something *isn't* there, *used to be* there, or *isn't being done* --
  "no libomp-dev here", "removed the X workaround", "we don't do Y
  anymore". A reader sees only the code that exists; a note about code
  that doesn't exist is unverifiable noise the moment they check, and dead
  weight forever after. If a line was dropped, dropping it needs no
  comment -- the absence speaks for itself. Only write a comment when it
  justifies something *present*.
- **Comments don't describe cross-component architecture.** If a comment
  needs to explain how this file relates to three other files, why a
  particular Dockerfile stage exists in the overall pipeline, or how the
  patch fits into the broader gfx803-vs-mainline story, that belongs in
  `README.md`/`MIGRATION_NOTES.md`/a patch header -- not in an inline code
  comment. A code comment's job is to explain the code immediately around
  it, not to teach the reader the whole system. If you're tempted to write
  three paragraphs above a function about how the pipeline works, that's a
  docs edit, not a code comment.

## Why this repo exists, and why it matters for how you work here

gfx803 (Polaris) is unsupported upstream since ROCm 6.0. Every fix here is
local -- nothing gets upstreamed to AMD, nothing gets fixed by a ROCm
version bump unless you go check. That has two consequences for how to
approach work in this repo:

1. **Assume nothing is fixed until you've checked the current pinned
   source.** A patch's own header saying "confirmed broken as of ROCm X"
   is a snapshot, not a permanent fact. Before re-diffing or investigating
   further, grep the current pinned commit for the target function/struct
   and read what's actually there now -- this repo's own history has
   examples both ways: patches that turned out to already be obsolete
   (the fix landed upstream on its own) and patches whose target code was
   *replaced* by something with unknown, unverified behavior on the same
   bug class (not fixed, not necessarily still broken -- genuinely
   unknown until checked).
2. **A patch that applies clean has confirmed nothing about correctness.**
   The recurring bug class on this hardware is silent miscompute --
   `rocblas_status_success` with wrong numbers, a kernel that dispatches
   fine and returns garbage. Re-diffing a patch so it compiles again is
   necessary but not sufficient; say so explicitly ("NOT YET RE-VERIFIED
   ON REAL HARDWARE") in the patch/notes until someone actually re-runs
   the original repro against the new binaries, not just confirms the
   diff applies.

## Investigation workflow (what's worked repeatedly in this repo's history)

1. **Trace before you patch.** `MIOPEN_ENABLE_LOGGING_CMD=1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Schaka/rocm-gfx803](https://github.com/Schaka/rocm-gfx803) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
