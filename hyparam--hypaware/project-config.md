---
trigger: always_on
description: HypAware is the active codebase. Prefer files under `src/`, `hypaware-core/`,
---

# Repository Guidance

HypAware is the active codebase. Prefer files under `src/`, `hypaware-core/`,
`bin/`, and root `test/`. The old `collectivus/` donor tree is not part of this
repo; do not assume its tests, package scripts, or agent notes are available
unless a task explicitly provides that context.

## Adding things

Make the smallest change that fixes the problem. When a bigger change looks
right, land the small one and defer the rest. File a GitHub issue autonomously
only when the follow-up is concrete, consequential, and clearly outside the
current task. Do not file issues for speculative improvements, minor cleanup,
or observations that are adequately captured in the PR description.

- **No new runtime dependencies.** Use the standard library and the code already
  here. If nothing here can do the job, keep the fix small and apply the same
  follow-up threshold above.
- **Do not invent columns, config keys, or schema fields.** Reuse or derive.
  Add one only when the task calls for it, and a rejected one stays rejected.
- **Reuse before you add** a file, helper, wrapper, or abstraction, unless the
  existing one is the wrong home for it.
- **Stop when tests pass.** Note unasked-for docs, cleanups, or adjacent fixes
  in the PR description; do not land them.

## Performance

Performance is a product requirement. Keep CPU work and memory use economical
and bounded, especially in hot paths, per-record work, and long-running
processes. Every code review must include an explicit CPU and memory pass over
the changed code and affected paths. Call out avoidable allocation, repeated
work, unbounded growth, busy loops, and behavior that worsens with data volume
or uptime; state explicitly when the review finds no CPU or memory concern.

## Design docs (LLP)

Design rationale lives in numbered **LLP documents** under `llp/`, following
Linked Literate Programming. Start at [`llp/0000-hypaware.explainer.md`](llp/0000-hypaware.explainer.md)
for the subsystem map, and [LLP 0002](llp/0002-v1-scope.decision.md) for what
actually shipped in V1.

- **Most changes need no LLP.** Bug fixes, null handling, tests, renames,
  behavior-preserving refactors, and version bumps get none. Write one when a
  real design decision is made or changed, not as a record of a fix.
- **Read before you change.** Before modifying a subsystem, read the LLP tagged
  with its `Systems` value (e.g. `Sources`, `Sinks`, `Plugins`, `Config`).
- **Annotate non-obvious decisions.** When you implement or change code that
  realizes a documented, non-obvious design decision, add an annotation:
  `// @ref LLP NNNN#anchor: short gloss` (with an optional relation before the
  colon: `[implements]`, `[constrained-by]`, `[tests]`, e.g.
  `// @ref LLP NNNN#anchor [implements]: short gloss`). Attach it directly above
  the construct; a blank line breaks attachment. Don't annotate mechanically; a
  ref must tell you something the code and filename don't.
- **Keep refs honest.** When you touch annotated code, check the referenced
  section still applies; update or remove the `@ref` if not.
- **Living docs.** Update the LLP when the design changes: land the doc edit in
  the same commit as the code. Mark retired docs `Superseded` or move them to
  `llp/tombstones/` with `Status: Tombstoned`; don't leave stale guidance.
- **Accepted docs are settled.** Once an LLP is `Accepted` or `Active`, do not
  edit what it settled. Change the design by extending it (a new LLP, noted on
  the old doc's `Extended-by:` line) or by replacing it with a new LLP and
  marking the old one `Superseded`. Mechanical edits are still fine: typos,
  broken links, status changes, and renumbering that does not change meaning
  (LLP 0156).
- **Tooling lives in-repo** under `.claude/skills/` (so every clone has it):
  `/ref-check [path]` validates `@ref`s; `/ref-story <file>` shows a file's
  rationale-order view; `/llp-create <title>` scaffolds a new doc; `/llp-list`
  surveys the corpus; `/llp-grill` stress-tests a plan against the LLP corpus
  before you write code.
- **A new number comes from `node scripts/llp-numbers.js next`**, after a
  `git fetch --prune`. Numbers are minted on every branch at once, so the tree
  you have checked out is not the corpus: three branches each read
  `max(llp/) + 1` and each got the same answer (issue #907). The same script
  gates it: `check` in the `cross-branch-numbers` CI job, which fetches every
  branch first, and in `npm test` wherever the clone carries them (a shallow or
  single-branch checkout skips it and says so). `survey` shows every collision
  across every ref.

## Code Style

- JavaScript, no semicolons.
- No em dashes (the U+2014 character) anywhere: code, comments, JSDoc, strings,
  or docs. In prose, use the punctuation the sentence wants (a comma, colon,
  parentheses, or a sentence split); in runtime strings, prefer `-`.
- No raw NUL bytes (U+0000) in tracked text. grep classifies a file holding one
  as binary and silently skips it, so the file drops out of every text search
  with no error to say so. When a string genuinely needs a NUL (a dedup-key
  separator, say), write the escape `\0`: it yields the same value and keeps the
  bytes on disk searchable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyparam/hypaware](https://github.com/hyparam/hypaware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
