---
trigger: always_on
description: Instructions for AI agents working in this repo. Humans may find the
---

# AGENTS.md

Instructions for AI agents working in this repo. Humans may find the
environment section useful too.

The default branch is **`master`** (not `main`).

---

## The rule: no change lands on `master` without an adversarial review

Before committing to `master` — including changes that look trivial, and
including changes that came with a detailed issue telling you exactly what to
do — you MUST run a **separate adversarial sub-agent** against the diff and act
on what it finds.

This is not a code-review formality. Its purpose is to catch the specific
failure mode that a single agent implementing a well-specified fix is worst at
noticing: the fix does what the issue asked, the tests pass, and it is still
wrong — because it was applied at the wrong layer, because another caller
reaches the same defect, or because the tests prove something the product does
not actually do.

### What the adversarial agent must be told

Instruct it to argue AGAINST the change and to assume the author is wrong until
proven otherwise. It must **run code to test its hypotheses, not speculate**,
and it must clearly mark anything it could not verify by execution. A fabricated
or unverified objection is worse than no objection, because it costs a real
investigation to dismiss.

At minimum it must attempt these theses:

| Thesis | The question it must answer |
|---|---|
| **Wrong layer** | Is this papering over a defect that belongs one level down? Enumerate EVERY caller — is there a path to the same bad outcome that bypasses this fix? |
| **Does not achieve the effect** | Construct real inputs that still produce the bad outcome after the change. Include the case where the fix *destroys* something a user wanted kept. |
| **Robustness** | Does the predicate hold at the edges? If a helper has multiple code paths (DOM vs. no-document, worker vs. main thread), do they agree — or do the tests pass on a path the product never takes? |
| **Performance** | Quantify, don't hand-wave. Is new per-item work being paid inside a loop that already runs at scale? |
| **Security** | Does this reorder anything relative to sanitization, permission checks, or rules enforcement? |
| **Test quality** | Are the tests vacuous? Do they exercise the product's real path? Does simplified test fixture markup hide the case that actually breaks? |

### Triage

The agent's findings must be sorted into **BLOCKING** / **NON-BLOCKING** /
**NOISE**.

- **BLOCKING** must be fixed before landing, or explicitly overridden by the
  repo owner in the conversation. Do not override it yourself.
- **NON-BLOCKING** is *first* checked against the rule below — fix it if it is
  small and unambiguous. Only what genuinely does not belong in this change
  becomes a GitHub issue, and it becomes one before landing, not a TODO comment
  and not a promise in a commit message.
- **NOISE** is dropped. Say so; do not silently ignore it.

### Leave everything a little better than you found it

**Do not defer cleanup that is unambiguous and well-scoped. Fix it now.**

Filing an issue is not free. It costs someone a second context-load to fix a
thing you were already looking at, with the file already open and the
reasoning already in your head. A one-line guard that makes a function match
its four siblings is not a follow-up; it is part of doing the work.

Fix it in the current change when **all** of these hold:

- The correct fix is obvious — no design decision, no choice between defensible
  options, no need for the owner's preference.
- It is small, and testable the same way the rest of the change is.
- It is in code you are already touching or already had to understand.

File an issue instead when any of these hold:

- It needs a decision (which sentinel, which key, which tradeoff).
- It is large enough to dominate the diff, so the change stops being reviewable
  as one idea.
- It reaches into a subsystem you have not established the invariants of.
- Fixing it would fan out — each fix exposing two more. **Scope is bounded by
  the change, not by the taste for tidying.** If cleanup starts recursing,
  stop, land what you have, and file the rest.

The specific thing to stop doing: noticing a defect *while fixing an adjacent
one*, in the same file, and writing it up instead of fixing it. Lint errors in
a file you are already editing, an unguarded access next to four guarded ones,
a comment your own change just made false — those are all "now", not "later".

When you do fix something extra, say so in the commit message and say why it
belonged there. A reviewer should never have to guess why an unrelated-looking
hunk is in the diff.

### The adversarial agent produces claims, not verdicts

Its output is a list of things worth testing. It is not a review you accept, and
"the adversarial agent said so" is not evidence. Before you act on a finding —
especially one that changes the shape of the fix — **verify it yourself, by
running it.** Subagents report confidently and are sometimes wrong; a finding
adopted on trust is indistinguishable from one you invented.

This cuts both ways, and the second direction is the one that gets skipped:

- Before **adopting** a recommendation, reproduce the evidence for it. If the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkomoros/card-web](https://github.com/jkomoros/card-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
