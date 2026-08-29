---
trigger: always_on
description: This file is the **operational playbook** for making large editorial changes to the book
---

# AGENTS.md — running editorial folds on the MAGE book with a fleet

This file is the **operational playbook** for making large editorial changes to the book
(`book/`) with dispatched agents. It extends `CLAUDE.md` §"Working on this repo with a fleet"
(single-live-writer + parallel drafting), which you should read first. Where `CLAUDE.md` states
the *rules*, this file gives the *worked process* — the one that lands a multi-part copyedit pass
in one sitting without serializing on `main` and without a merge that fights 100 generated files.

The method has four patterns. Use them together: **draft in parallel → fold in parallel worktrees →
merge by source-checkout → resolve the model layer once at the end.**

---

## 0. The one fact everything follows from

`main` has a single-live-writer rule for exactly one reason: the `pre-commit` hook rebuilds the
site and **force-stages the regenerated `.html` + derived `book-models/*`** on every commit, so two
agents committing to the *same branch* collide on those generated files. **Everything else about the
book is deterministic from source.** That single fact is what makes the patterns below safe:

- Two folds on **separate worktree branches** never collide (separate git indexes).
- Two folds touching **disjoint source trees** (different Parts / the appendices) never conflict on
  content — only on regenerated files, which you never merge, you **rebuild**.
- A **PDF renders from the working tree**, not from `HEAD` — so you can build a review PDF from
  staged-but-uncommitted source.

---

## 1. Pattern — disjoint-worktree parallel folds

When several folds touch **disjoint source trees** (e.g. Part II prose, Part III prose, appendices),
run them **concurrently in separate git worktrees** instead of serially on `main`.

**Setup, per fold:**
1. `git worktree add -b <fold>-fold <path> HEAD` (branch from current `main`; put `<path>` in a
   scratch dir).
2. ★ **Copy the drafts into the worktree.** `book/_design/drafts/` is **gitignored**, so a fresh
   worktree checkout does *not* contain the EDIT-MAP or the authored figures. Without this the agent
   has nothing to fold:
   `cp -r book/_design/drafts/<effort>-<date> <worktree>/book/_design/drafts/`
   (they stay untracked in the worktree — the hook never stages them).
3. Dispatch the fold agent with its **CWD = the worktree** and a brief that says: you are on branch
   `<fold>-fold`, confirm with `git branch --show-current`, **do NOT touch `main`**, other folds run
   concurrently on disjoint trees — ignore them.

**Before you trust "disjoint":** `git diff --name-only main...<otherbranch>` and confirm no overlap
on **source** (exclude `.html` and the flip-floppy derived book-models — see §3). Shared
*hand-authored* book-models (`figure-caption-tiers.json`, `chapter_identity/shape`, `index-terms.md`)
are the only real overlap surface; have each agent **report** which of those it touched.

---

## 2. Pattern — the standard copyedit-fold brief ("just land the copyedits")

When the job is an editorial/copyedit pass and the goal is to **land prose fast for review**, not to
perfect the model layer, give the agent this contract:

> **Don't worry about cross-coherence or the model layer — just APPLY the edits per the EDIT-MAP and
> commit through the hook.** The prose is authoritative (it's the just-reviewed copy). Where a model
> or registry disagrees with the landed prose, that's a *deferred* item, not a reason to touch prose.

Concretely, the brief must say:
- **Gate cadence = the fast hook only.** Per commit, just `git commit`; the `pre-commit` hook runs
  `catalog.py validate` (~7s) + `build` + the `book-float-ref` gate. That is the correctness net.
  **Do NOT run `catalog_tests.py`** (the ~238s model suite) and **do NOT self-verify** any
  model/suite-level check. Model resolution is a separate batch-repair (§4).
- **Commit early and often** — per `§`-section or per appendix. The API is flaky; never hold a large
  uncommitted batch. (A prior fold died on an API drop and lost an uncommitted batch — commit-per-
  section makes that a non-event.)
- **Guardrails:** NEVER `git add -A` (sweeps `book/_design/`), NEVER `--no-verify` (skips the
  force-stage), never stage `book/_design/` or `scratchpad/`. Carry the co-author trailer.
- **Report the shared files it touched** (the book-models in §1) so the merge can hand-union them.

If a genuinely load-bearing scope question arises (a brief contradicts an EDIT-MAP, two lanes might
own the same figure), the agent should **defer the call to the orchestrator**, not guess — a wrong
guess on a shared file becomes a merge conflict.

---

## 3. Pattern — the merge (source-checkout, never `git merge`)

A real `git merge` of a fold branch conflicts on ~100 regenerated `.html` files. Don't. Land the
branch's **source only** and let the hook regenerate everything derived:

1. **List the branch's source files** (exclude generated + flip-floppy derived):
   `git diff --name-only main...<branch> | grep -vE '\.html$|book-models/(projection-index|reverse_index|outline|outcomes|chapter_identity|metaphor-slogan-index)\.json$|book-models/models-view\.html$'`
2. **Confirm disjointness** vs what `main` gained since the branch point:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davisjam/model-based-agentic-software-engineering](https://github.com/davisjam/model-based-agentic-software-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
