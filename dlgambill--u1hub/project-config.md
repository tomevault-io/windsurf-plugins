---
trigger: always_on
description: Log mistakes in `MISTAKES.md` (what happened, root cause, prevention). Newest
---

# CLAUDE.md — U1 Print Hub

## Mistake log

Log mistakes in `MISTAKES.md` (what happened, root cause, prevention). Newest
first. Read it before touching an area you have broken before. When the same
failure appears 4–5 times, promote it to a hard rule below.

**Only mistakes made in THIS repo.** `MISTAKES.md` is committed to a public
repo, and `SESSIONS.md` sends every lane here at startup whatever repo it is
working in. If you are not the `u1-print-hub` lane, your mistake goes in
`C:\Users\Danny\code\MISTAKES-shared.md` - never here. This has now happened
twice and been undone by hand both times (SESSIONS.md thread 6, and the
2026-09-10 02:50 UTC entry, where a foreign entry was sitting uncommitted
under a release commit). The same applies to the promoted-cluster table: the
cross-project clusters are counted in `MISTAKES-shared.md`, which keeps its
own table and its own sequence. The paragraph under "Where durable
instructions belong" said this already; it is repeated here because this is
where a lane is standing when it decides where to write.

## Other sessions on this machine

Danny runs more than one Claude session against ichabod at once, and they
cannot see each other. **Read `C:\Users\Danny\code\SESSIONS.md` after this file
and `MISTAKES.md`, whatever repo you are working in.** It carries an ownership
register for shared resources — which session owns which repo, table and
service — plus open questions between sessions and their answers.

Append to it (never rewrite) when you take a shared resource, answer another
session's question, or find something that affects a repo you do not own. It
exists because two sessions were writing to `dlgambill/conduitlab-site` for
months and neither knew.

## Visual standard

**Read `C:\Users\Danny\code\DESIGN.md` before building or changing anything with
a user-facing surface**, whatever repo you are in. It is the VibeCurb
constraint approach (github.com/Yu-369/VibeCurb) Danny wants applied to every
project instead of framework defaults: extract the design signals before writing
code, refuse the banned defaults (CSS keyword easings, AI-purple gradients,
generic dashboard layouts, weak typographic hierarchy), and verify the result
against the extraction rather than against a general impression.

It also records which VibeCurb skills are actually installed on the account, so
you do not offer one that cannot be invoked.

## Where durable instructions belong

Put anything a future session must know in a FILE that this file points to —
never in a scheduled task's prompt. A task with computer access can only have
its prompt changed by Danny, in a conversation running inside the Claude desktop
app on ichabod, and he is usually not at ichabod. Files need no approval and
take effect on the next session. This section exists because a session tried to
add SESSIONS.md and DESIGN.md to the `ichabod on demand` prompt, hit
`needs_device_approval` twice, and only then noticed the prompt already reads
this file.

**There is one tree: `C:\Users\Danny\code\u1-print-hub`, this git clone.**
Edit here, run the Hub from here (port 4545), commit here. Danny retired the
`X:\u1-print-hub` staging copy on 2026-09-09 after the two-tree arrangement
cost real bugs (rfid.js shipped five releases stale; this file diverged in both
directions in one week). The gcode library moved to **`X:\gcode`** and
`config.json` points at it; everything else the Hub remembers (config, spools,
slots, schedule, password, tunnel, thumbnail cache) sits in this directory,
gitignored. `scripts/sync-to-clone.js` and `scripts/drift-scan.js` are stubs
that say so. If `X:\u1-print-hub` (or a `_RETIRED` rename of it) still exists,
it is dead weight waiting for Danny to delete it - never read from it.

`MISTAKES.md` is this repo's public log, so a mistake made in another project
goes in that project's log or in `C:\Users\Danny\code\MISTAKES-shared.md`, not
here.

## Hard rules

1. **Rule #1 — read before writing.** Read every file you are about to change,
   in full, before the first edit. No patching from memory or from a grep hit.
   **And read what it loads.** A file's behaviour is the whole chain it pulls
   in — resolve every `<link>`, `<script src>`, `require` and `import` before
   concluding anything, especially before claiming something is absent. An
   "there is no X here" claim is a whole-project claim; a count from one file is
   not evidence for it. (2026-09-01: audited index.html, missed the gold.css it
   links one line later, and reported a stylesheet that already existed as
   missing.)
2. **Nothing ships unverified.** "Built" is not "shippable." A feature is done
   when the harness is green *and* its live hardware gate has passed. Unverified
   shapes and paths are not emitted at all.
3. **This clone is the only tree** (since 2026-09-09; see above). Code is read,
   edited, run and committed in `C:\Users\Danny\code\u1-print-hub`. Never
   build or reason from GitHub main instead of the working copy, and never
   from a retired staging folder on `X:`. The gcode library is `X:\gcode`; the
   Hub's state files live beside the code and are gitignored. The `X:` share
   refuses rename-over-existing, which is why state writers use the fallback

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dlgambill/u1hub](https://github.com/dlgambill/u1hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
