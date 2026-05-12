---
trigger: always_on
description: PR economics inverted when AI became the reviewer:
---

# Agent Instructions

## 500-line hard limits

PR economics inverted when AI became the reviewer:

- **Human reviewer**: 10 small PRs = 10 context switches → prefers big PRs.
- **AI reviewer**: 1 big PR > context window → prefers small PRs.

This codebase optimizes for the AI reviewer. The 500-line ceiling is the
forcing function that makes that work.

Direct consequence: **each commit is a PR**. Continuous integration's
real form when the reviewer is an AI is "every mergeable change ships
on its own". 30 minutes of coding → commit → PR → AI review → merge →
next. No batching. No save-up-for-Friday-review meeting. Human
programmers find this annoying; AI co-authors thrive on it.

### The two budgets

- **No `.rs` source file exceeds 500 lines** of *production* code.
- **No PR diff exceeds 500 lines** of *production* code.
- Both limits derive from one constraint: AI co-authors (Codex,
  Copilot, Claude) cannot reliably hold more than ~500 lines of context
  at once. Past that they hallucinate, contradict prior parts of the
  same file/PR, or silently skim.
- **Test code does not count toward either budget.** `#[cfg(test)] mod
  tests { ... }`, `#[tokio::test]` blocks, and integration tests under
  `tests/` are stereotyped: each test is independently legible, and a
  reviewer scans them top-to-bottom rather than holding the whole
  module in working memory. A 200-line production module with 600
  lines of tests is one self-contained reviewable unit, not a budget
  violation. (Reference precedent in this codebase:
  `sdk/tests/e2e_blackbox.py` is several thousand lines and reviewed
  one assertion at a time without trouble.)
- **Slight overage of the production budget is acceptable when the
  maintainer has read the change in full and explicitly signed off.**
  The budget is "AI working memory", not arithmetic — 510-550 lines
  with a human in the loop is fine, 1500 lines is never fine. The
  hard ceiling is "an AI agent can still hold the whole production
  surface at once"; exact threshold is judgment.
- Exceeding the production budget without sign-off requires splitting
  before review.

#### How to count

For most files: total `wc -l` minus the `#[cfg(test)] mod tests { ... }`
block. The block is contiguous and at the bottom of the file by
convention, so a quick `grep -n '^#\[cfg(test)\]' core/src/foo.rs`
gives the start line; the file's total minus that line number is
the production count (off by one or two for the trailing brace —
fine, the rule is judgment, not arithmetic).

When the production count is non-obvious or close to the limit,
paste it into the PR description so reviewers don't have to
re-derive it. For files with both inline `#[cfg(test)]` snippets
and a bottom `mod tests`, sum the production code by reading.

### Diff-only review

Reviewers see only the diff, never the surrounding file. This is the
optimal use of an AI context window: don't reload context that didn't
change.

Concrete consequence: **cascading PRs are the natural form, not a
workaround**. PR N's base branch is PR N-1, not master. Each PR's diff
is one self-contained increment. The reviewer never sees PR 0's lock
change while reviewing PR 4's pipeline extraction; only the pipeline
extraction.

```
master
└─ PR 0 (10 lines)
    └─ PR 1 (300 lines, base = PR 0)
        └─ PR 2 (150 lines, base = PR 1)
            └─ PR 3 (300 lines, base = PR 2)
                └─ PR 4 (500 lines, base = PR 3)
```

Without cascading, PR 4's diff = PR 0 + 1 + 2 + 3 + 4 = 1260 lines = AI
loses the thread. With cascading each PR is an independent 500-line
review.

### Cascade stack depth: 3-4 levels max

Cascading is not free. To review PR N a reviewer (human or AI) has
to first mentally accept PR 0 → PR 1 → ... → PR N-1, then read PR N's
diff against that imagined state. **Each level adds one item to the
mental stack.** Past 3-4 levels the stack overflows for the same
reason a single 1500-line PR overflows: working memory runs out.

The cascade form does not eliminate the budget; it changes what the
budget is spent on. Per-PR diff stays at 500 lines, but cumulative
stack-depth context is bounded too.

**The rule**:

- **Soft cap: 3 levels**. Comfortable.
- **Hard cap: 4 levels**. Acceptable when the bottom levels are
  small or trivial (e.g., one-line `chore/visibility-fix`).
- **Above 4: stop adding new PRs. Drain the stack first.**

**Draining**:

1. Merge the **bottom** of the stack (the level closest to master,
   typically the first-written PR) into master.
2. The level above it now has its base auto-redirected to master and
   becomes depth 1.
3. Keep merging upward until the stack is shallow enough.
4. Then resume opening new PRs.

```
Before drain:
  master → PR 0 → PR 1 → PR 2 → PR 3 → PR 4 → PR 5
                                              (depth 6, blown)

After merging PR 0, 1, 2:
  master(includes 0/1/2) → PR 3 → PR 4 → PR 5
                                              (depth 3, fits)
```

This is why the merge order matches the cascade order: oldest /
deepest-base first. Trying to merge a higher-up PR while a lower-down
PR is still open creates a divergent base that GitHub will not
auto-redirect cleanly.

### Pure-mv PRs

A PR that mechanically moves N lines from one file to another counts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rangersui/Elastik](https://github.com/rangersui/Elastik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
