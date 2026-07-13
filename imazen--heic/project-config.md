---
trigger: always_on
description: **While working on a multi-step task in this repo, you do not stop, pause, lazily
---

# HEIC Decoder Project Instructions

## ABSOLUTE MANDATE: NEVER STOP PUSHING FORWARD

**While working on a multi-step task in this repo, you do not stop, pause, lazily
ask "shall I continue?", or wrap up the session early — unless one of the
genuine stop conditions in `~/.claude/CLAUDE.md` ("NEVER PAUSE LAZILY") fires.**
The default is: ship the next chunk. Then the next. Then the next.

**Stopping criteria — these are the ONLY legitimate reasons to pause:**

1. The next step is **destructive or externally visible** (force-push, publish,
   posting outside this org, deleting shared state) — confirm per the global
   "Executing actions with care" rules.
2. The next step **branches in a way that's not derivable from prior
   instructions** — two equally-valid paths with no expressed preference.
3. You hit a **measured-and-verified** blocker (no GPU, no Apple HW, missing
   library version not on crates.io) and there is no chunk you can ship
   that doesn't depend on that block.
4. Native-API runtime tests across all six backend targets pass, AND every
   item on the user's last list is shipped or has a documented chunk that
   cannot be shipped today.

**Forbidden — these are NOT stop conditions:**

* "I've done a lot already, let me summarize."
* "Want me to keep going?"
* "Should I continue with X, or hold?"
* "The next step is heavy."
* "Compile takes a while."
* "CI is yellow on one job."
* "I'm not sure if you want me to..."
* End-of-session-feeling because the conversation is long.

If a chunk is genuinely too large to ship in one pass, decompose into the
smallest demoable chunk (per the global "NEVER GIVE UP ON A USER-DIRECTED
LIST" rule), land that chunk with a passing test, document the next chunk
with file paths + signatures, and **keep going on the next chunk** in the
same turn. Do not stop to ask.

## Writing Good Code — patterns imported from jxl-encoder

These patterns are mandatory reading and apply to every commit. Adapted from
`~/work/zen/jxl-encoder/CLAUDE.md` "Patterns of Mistakes to Avoid" + "Proof-
by-Tests Investigation Methodology" + "Invariant Preservation Across Sessions".

### 1. False positives are the highest-severity bug

Tests that pass without exercising the thing they claim to test are worse
than no tests — they manufacture false confidence and waste future
investigation time. For a decoder, "the parser accepted the bytes" is
**not** evidence "the image decoded correctly" — you must call all the
way through `decode_to_frame` / `to_rgba` / the backend's `decode_hevc`
and inspect actual pixels. The `mf_diff` example + zensim-regress corpus
diff are the canonical templates for this codebase.

Rules:
- **Never** declare a backend "works" based on `is_available()` alone.
  Decode example.heic via the backend and verify dimensions + pixel
  equivalence (zensim ≥ 95) against the rust backend.
- **Never** trust a test-count delta. Verify what the tests actually do.
- When fixing a "tests pass but it's wrong" bug, audit every other test
  that uses the same pattern.

### 2. Read existing docs before investigating

Before "investigating" any bug, read:

1. This file's "Known Bugs" and "Investigation Notes" sections.
2. `CHANGELOG.md` `[Unreleased]` and the most recent shipped version.
3. `git log --since="3 days ago" --oneline -30`.
4. `git log --grep="<error fragment>" --all`.
5. The relevant backend's `PORTING.md` if applicable.

If the bug is already documented, continue the existing investigation —
do not start a new note. Update in place.

### 3. Test the test infrastructure

When you add a test helper (corpus harness, fake fixtures, diff utility),
exercise it against known-good and known-bad inputs before relying on it.
A test helper bug is worse than the bug it's trying to catch — it
poisons every test built on top.

### 4. Documentation reflects what's verified, not what's intended

Before claiming a feature "works" in CLAUDE.md, README, CHANGELOG, or a
commit message:

- Decode a real HEIC end-to-end via the path being claimed.
- Compare against libheif / the rust backend / dec265 — at least two
  external sources of truth.
- For native backends, the bar is `compare_backends_via_zensim` reporting
  zero failed files in the bundled corpus.

Status markers (used in CLAUDE.md + CHANGELOG):

| Marker | Meaning |
|---|---|
| ✓ Complete | Works end-to-end, ≥ 2 cross-checks, runtime CI green. |
| ⚠ Partial | Some inputs work, others fail; failure mode documented. |
| ⚙ In Progress | Implementation exists, not yet exercised against real input. |
| ✗ Broken | Implementation exists, known-failing test pinned. |
| ❌ Not Started | No implementation. |

### 5. One commit, one complete fix

Multiple `fix: correct X` commits for the same `X` within a day means the
first fix was shipped without understanding. Before fixing a bug:

- Trace every consumer of the wrong data.
- Write a failing test that reproduces it.
- Understand **why** the bug exists, not just where.

After: verify with a different code path (e.g. the MF backend fix
verified via both `mediafoundation_alone_decodes_when_required` and the
corpus zensim diff).

### 6. Investigation lives in ONE place

CLAUDE.md "Investigation Notes" is the single source of truth. Do NOT

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imazen/heic](https://github.com/imazen/heic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
