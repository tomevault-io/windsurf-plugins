---
trigger: always_on
description: This file documents the agentic workflow used to develop LLVM-in-Rust with
---

# AGENTS.md — Agentic Development Guide

This file documents the agentic workflow used to develop LLVM-in-Rust with
coding agents.  It exists so that agents can operate **autonomously** on this
project with minimal back-and-forth, following the patterns established across
the production-readiness roadmap (issue #93, Milestones A–Z).

---

## Standing Roadmap-Execution Workflow

This is the durable process for advancing the production-readiness roadmap
(**issue #93**).  It is the default mode of work — agents should follow it
without waiting for a per-task restatement of these rules.

1. **Source of truth is #93.** Pick up the lowest-lettered milestone that is
   still open (its tracking issue is linked from #93).  Milestones are worked
   roughly in order; later milestones may depend on earlier ones being green
   (e.g. the RC burn-in milestone is blocked until all prior milestones pass).
2. **One issue → its own focused PR(s).** Solve each tracking issue (or each
   self-contained checklist item within a large milestone) in a separate PR.
   Keep mechanical changes (fmt, clippy, renames) in their own PRs, separate
   from behavioral changes, so each diff stays reviewable.
3. **Every PR carries thorough tests.** Add regression tests that would have
   caught the bug or that exercise the new behavior.  Prefer
   target-independent tests (assert on IR / data structures) so they run on
   every platform, not just the host arch.
4. **Code review is mandatory** (see the loop below).  When two agents
   collaborate, divide milestones explicitly up front so branches never
   overlap, and review each other's PRs.  GitHub blocks self-approval when
   pushes share one account, so post the review as a PR **comment** with an
   explicit verdict instead of a formal approval.
5. **Quality gates must stay green** on the merge commit: `cargo +stable fmt
   --all -- --check`, `cargo +stable clippy --locked --all-targets -- -D
   warnings` (both enforced by `.github/workflows/quality-gates.yml`), the full
   test suite, the Differential Tests workflow, and the Platform Matrix gate.
   Do not mark any production-readiness status green unless all of these agree
   on the same commit.
6. **Close the issue after its PR merges** and tick the corresponding checkbox
   in #93; keep the #93 status table current.
7. **Report once a milestone is done** in the active coordination thread and
   update #93 with links to the green CI runs.
8. **Auto-resume across interruptions.** Keep `MEMORY.md` (in the agent
   workspace) current as the recovery point: which milestone is in flight,
   which PRs are open, and what is next.  After a rate-limit, sleep, or daemon
   restart, re-sync `origin/main`, re-read #93, and resume from the lowest open
   milestone — no human nudge required.

---

## Development Lifecycle

Every feature follows this six-stage cycle, executed end-to-end without
user prompts at each step:

```
Plan → Implement → PR Review → Test → Issue+Fix Loop → Merge
```

| Stage | Slash skill | Description |
|-------|-------------|-------------|
| Implement a phase | `/implement-phase` | Branch → code → targeted tests → commit → PR |
| Review implementation PR | `/review-and-fix` | Review diff/tests → run full tests → post PR feedback |
| Fix one issue | `/fix-issue <N>` | Read issue → fix in same PR branch → test → update PR |

### Mandatory PR Review/Test/Issue Loop (for implementation PRs)

Before merging an implementation PR, the agent must:

1. Review the PR diff and changed tests with a code-review mindset (correctness, regressions, missing tests).
2. Run targeted tests plus a full test sweep (`cargo +stable test` unless blocked).
3. If concrete problems are found, open GitHub issue(s) documenting them.
4. Fix those problems in the **same PR branch** and push follow-up commits.
5. Post PR feedback (`gh pr review --comment` or `gh pr comment`) summarizing findings, linked issues, and fixes.
6. Merge only when checks are green and no unresolved review findings remain.
7. After merging, verify and close the associated GitHub issue(s) referenced by the PR.

---

## Git Workflow Conventions

These rules prevent common mistakes in the multi-worktree setup:

| Rule | Reason |
|------|--------|
| Always branch from `origin/main`, not `main` | `main` is checked out in the primary worktree; checking it out again fails |
| `gh pr merge <N> --squash` (no `--delete-branch`) | Same worktree conflict |
| Stage specific files, never `git add -A` | Avoids accidentally committing `target/` or secret files |
| Never use `--no-verify` | Fix the hook failure instead |
| Run `cargo test` before every commit | All tests must be green |
| If review finds bugs, open issue(s) and fix them in the same PR branch | Preserves traceability and keeps context in one PR |
| Post at least one PR review feedback comment before merge | Captures reviewer reasoning and findings in GitHub history |
| Always verify and close associated issue(s) after merging a PR | Keeps GitHub project state aligned with merged work |

**Branch naming:**
- Features: `feature/phase<N>-<slug>` (e.g. `feature/phase4-x86-backend`)
- Fixes: `fix/issue-<N>-<slug>` (e.g. `fix/issue-30-mov-to-preg`)

---

## Agent Usage Guide

### rust-stable-compat agent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yudongusa/LLVM-in-Rust](https://github.com/yudongusa/LLVM-in-Rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
