---
trigger: always_on
description: This file tells the assistant how to work in `lean-interact`. It is not background reading;
---

# CLAUDE.md — operating instructions for this repository

This file tells the assistant how to work in `lean-interact`. It is not background reading;
it is the procedure. Follow it literally.

The short version: **the user states mathematics, you produce compiled Lean, fast, and you
never assert a Mathlib name you have not verified.** Everything below serves those three
things.

---

## 0. Start of every session

Before responding to the first mathematical request, read, in this order:

1. **`MEMORY.md`** — durable facts and preferences. Non-negotiable context. It tells you who
   the user is, what the environment guarantees, and which rules have been established. If
   its template sections are still unfilled, assume the default this project was built for: a
   working mathematician who knows the mathematics and is learning Lean and Mathlib. Then ask
   the user to fill those sections in, once, and record the answers.
2. **`vault/Formalization Style.md`** — how Lean is written here: naming, statement shape,
   tactic style, `ℕ` vs `ℤ` vs `ZMod n`. Entries marked PROVISIONAL are the project's guess,
   not the user's stated wish; follow them, but never defend them. If the user objects, the
   user is right and the note gets updated.

Then skim the top entry of `JOURNAL.md` for where the last session stopped.

Do not skip this because the request looks simple. The whole point of those files is that
they contain decisions you would otherwise re-litigate or silently contradict.

If a live session is not already running, start it: `tools/session.sh` (see `--help`).

Five skills package the recurring procedures, and you should use them rather than
improvising: **`install`** (bring a fresh clone to a working state — the precondition for all
the others), **`formalize`** (prose in, theorem out), **`formalize-from-magma`** (computation
in, theorem out), **`mathlib-lookup`** (find and verify a name), **`lean-session`** (bring up
or repair the live views).

---

## 1. The formalization loop

The user types a sentence of ordinary mathematics. Your job is the following sequence, every
time.

**Step 1 — Restate the sentence precisely.**
Before any Lean, say back what you understood, in mathematics, with every quantifier,
hypothesis and side condition made explicit. Ordinary prose leaves things implicit ("let `p`
be prime" often silently means odd; "`a/b`" often silently means `b` divides `a`). Surface
those choices, and when it matters, ask. A formalization of the wrong statement is worse than
none, because it looks like progress.

**Step 2 — Write the informal record.**
Put the user's sentence and its LaTeX form into `Scratch/Current.md`. This is what lets
anyone later check the Lean against the intent, and it is what the live views display beside
the code.

**Step 3 — Find and verify every name you will need.** See section 2. Do this *before*
writing the proof, not after the error.

**Step 4 — Write the Lean.**
Statement and proof into `Scratch/Current.lean`. Import **`NtLean.Preamble`** (section 3).
Remember `autoImplicit = false` and `relaxedAutoImplicit = false`: **every binder must be
declared explicitly**, type variables included. A missing binder is a hard error here, not a
convenience.

**Step 5 — Let the watcher compile.**
Saving is enough: `tools/live.py` polls the mtime, runs Lean through the daemon, and writes
`.live/status.json`. Read the result from that file rather than running Lean yourself while
the watcher is up. To check a fragment without disturbing the live file, use
`tools/leanserver.py snippet` — it goes through the warm daemon; `tools/leanlib.py`'s
`lean_snippet` always takes the slow one-shot path.

**Step 6 — Iterate to zero.**
Keep going until `state` is `"ok"`: **zero errors and zero sorries**. A `sorry` is a
legitimate intermediate step, and a legitimate way to show the user exactly where the
difficulty sits, but it is never an endpoint and never something left behind quietly. If a
`sorry` survives to the end of a turn, say so in the first sentence of your reply.

**Step 7 — Promote.**
Move the result into the right module under `NtLean/`, add the import to `NtLean.lean` if the
module is new, and run `lake build` to confirm it integrates. Promotion is the only moment
`lake build` is used. **After it, restart the daemon** (section 3) — otherwise the next check
of `Scratch/Current.lean` will not see the new declaration.

**Step 8 — Write the vault note.**
One note per promoted result, from the template in `vault/Templates/`. It carries the
informal sentence, the LaTeX, the final Lean, the lemmas used, and `[[wikilinks]]` to related
notes. Navigation between notes is a goal of the vault, so the links matter; a dead one is
invisible in Obsidian until someone clicks it, which is why `tools/check_names.py` also
checks them.

**Step 9 — Append to the journal.**
One bullet in the top section of `JOURNAL.md`: what was proved, in which module, and why it
mattered. Create a section for today's date if there is none. Failed attempts get a bullet
too, with the reason — that is usually the most valuable line in the entry.

### Turn length, and deferred bookkeeping

This is a live, conversational instrument. **A turn should take seconds.** Steps 1 to 6 are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nasqret/lean-interact](https://github.com/nasqret/lean-interact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
