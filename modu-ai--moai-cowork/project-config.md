---
trigger: always_on
description: Every clause here binds a turn regardless of which agent harness drives it. The file is
---

# AGENTS.md — standing contract for agents in this repository

Every clause here binds a turn regardless of which agent harness drives it. The file is
**self-sufficient**: it assumes no other instruction file is loaded, and no nested `AGENTS.md`
exists anywhere in this repository.

**Budget warning.** A personal `~/.codex/AGENTS.md` joins the same merged chain and is consumed
**before** this file, narrowing what the project's contract can carry. Overflow is dropped from the
**tail**, silently — no warning, no stderr, exit 0. Clauses below are ordered most-critical-first
for that reason.

Obligations are carried from `.claude/rules/moai/**` and `CLAUDE.md`, which remain the source of
truth; compression removed rationale and incident records, never an obligation. Claude-only
mechanisms (the question channel, subagent spawning, skills, session handoff) stay there.

---

## 1. Evidence and verification claims

**No unobserved claim.** An actor MUST NOT assert a verification, a completion, **a defect / debt /
drift, OR the premise underlying a recommendation** it did not actually verify with the domain's
mechanical tooling. Evidence absent is not evidence of success — nor of failure. The absence of a
failure signal never establishes that a check passed; a text-pattern inference is a hypothesis, not
a verified defect; a reference existing does not establish that the referenced capability is still
live. Reachability is not justification.

**Baseline-integrity attribution.** Every verification claim MUST be attributed to an
actually-measured baseline — the command that was run plus the output observed, in this run,
against this tree. A figure carried over from another package, tree, or point in time is not a
baseline; using it as a fresh measurement violates this. Anything unattributed is a Gap, not a
Claim.

**Evidence-bearing report format.** Verification and completion reports SHOULD carry five sections,
on every report and not only the first: **Claim** (what is asserted); **Evidence** (the command run
plus its verbatim output — a summary is not evidence); **Baseline-attribution** (what it was
measured against, in this run); **Gaps** (what was explicitly NOT observed — an empty Gaps section
asserts nothing was left unobserved, which must itself be true); **Residual-risk** (what could
still be wrong despite what was observed).

---

## 2. Git, branches, and the shared checkout

The primary checkout is shared — several sessions may work in it at once, and branch state there is
global.

**Never change branch state in the primary checkout.** Forbidden there: `git checkout <branch>` /
`git switch` (relocates every concurrent session's tree); `git checkout -b` / `git switch -c` /
`git branch` (same, plus an unexpected branch); `git reset --hard` / `git checkout -- <path>`
(discards work of unknown provenance); `git stash` (repository-global — it silently absorbs another
session's uncommitted changes); `git rebase` / `git merge` onto the checked-out branch (rewrites or
advances shared history mid-operation). Read-only inspection, `git fetch`, commits to the
already-checked-out branch, and pushing it are permitted.

**Re-read branch and commit state immediately before any commit or push** — never a value read
earlier in the turn, never the branch reported at session start:

```bash
git rev-parse --short HEAD
git branch --show-current
```

A difference from what the turn assumed means another actor is writing the same tree: stop and
report the divergence instead of proceeding.

**Never sweep-stage.** In the primary checkout, never `git add -A`, `git add .`, or
`git commit -a`. Stage by explicit pathspec and re-read `git status --short` immediately before
staging, so another session's files are visible and excluded. This binds **even when no foreign
session was detected** — one can arrive after the check, and the sweep is what turns its presence
into lost work.

**Detect parallel sessions before a non-trivial direct edit** to a shared path (`.claude/`,
`.moai/`, `internal/`, `pkg/`, `cmd/`, repo-root config), and surface any divergence:

```bash
git fetch origin main 2>&1
git rev-list --count --left-right origin/main...HEAD
```

`0 0` or `0 N` proceeds; `N 0` or `N M` means resolve before editing. Where another live session
shares the checkout, isolate into a worktree rather than editing in the shared tree. The check
decays — re-run it before any commit and after a long pause.

---

## 3. Worktrees

**Work inside a worktree, entered through the launcher** (`moai cc -w <name>`,
`moai cc -w <name> --spawn` for a new window, `EnterWorktree(<path>)` to re-enter); never create one
with a bare `git worktree add`. Leave with `ExitWorktree`. Drive a worktree with `git -C <path>`,
not `cd`.

**`moai worktree done` closes L2 trees only.** A tree under `.claude/worktrees/` is L1, is absent
from the registry, and is disposed by the session-end prompt or by `git worktree unlock` +
`git worktree remove`.

**A card's branch is unpushed, so its worktree holds the only copy of the work.** Dispose of no
worktree — L1 or L2 — until the branch is integrated and the remote merge has landed.

**Start a new card in a new worktree.** Exit any previous worktree back to the primary checkout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modu-ai/moai-cowork](https://github.com/modu-ai/moai-cowork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
