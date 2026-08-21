---
trigger: always_on
description: This tells any Claude agent (and humans) how work is planned, tracked, and shipped in
---

# CLAUDE.md — How we track and execute work in this repo

This tells any Claude agent (and humans) how work is planned, tracked, and shipped in
**VibeHarness**. Read it before starting a task.

> Repo-wide process doc — kept identical on `main`, `beta`, and `beta_mythos_fast`.

## TL;DR
- **GitHub Issues are the single source of truth.** Use the `gh` CLI. Every feature, bug,
  analysis, and test is a GitHub issue.
- **Work is staged across dependent issues:** Analysis → Development (fix/feature) →
  Testing / Re-verification — each its own issue, chained by explicit dependency references.
- **One branch (or git worktree) per issue.** A PR links to and closes its issue and is
  **reviewed by the main (orchestrator) agent** before merge.
- **The main agent orchestrates; subagents execute.** The orchestrator files & chains
  issues, dispatches subagents, enforces stage gating (don't start a stage until its
  prerequisite issue/PR is merged), and reviews PRs. It implements little directly.

## 1. Issue tracking (GitHub Issues + `gh`)
- Labels in use: `bug`, `enhancement`, `documentation`, `question`, `duplicate`, `wontfix`,
  `invalid`, `help wanted`, `good first issue`.
- Browse / create:
  - `gh issue list --state all` (or `--state open|closed`)
  - `gh issue view <n>`
  - `gh issue create --title "…" --label <label> --body "…"`
- Body convention (match existing issues): open with `## <Type>: <Title>`, then context, a
  spec / ground-truth section, and an **acceptance checklist** (`- [ ]`). Cross-reference
  related work inline as `#NN` / `(PR #MM)`.

## 2. Staged, dependency-chained issues
Stages are tracked as **separate** issues that reference each other. Title patterns actually
used in this repo:

| Stage | Title pattern | Examples |
|---|---|---|
| Analysis | `Analysis: …` / `Analysis FIRST: …` | #76, #71, #61 |
| Fix / Development | `Fix: … (Dependent on #NN)` | #77 (dependent on #76) |
| Feature | `Feature: …` | #22, #24 |
| Test / Re-verify | `Dependent test: …`, `Re-verify (live): …`, `Test: …` | #89, #58–#60, #21 |
| Follow-up | `Follow-up (#NN): …` | #92, #99, #100, #113 |

- Express the dependency in the body: **`Dependent on #NN`** / `gated on #NN` /
  `(depends on the analysis)`. A dependent stage does NOT start until the prerequisite
  issue (and its PR) is complete and merged.
- Worked chains: **#76** (analysis: leaked `llama-server` runners) → **#77** (fix,
  *Dependent on #76*). And **#105** (align prompts/schema to mythos_fast) → **#106**
  (cat-search YouTube test, *gated on #105*) → **#116** (validate & verify alignment).

## 3. Branches, worktrees & commits
- **One branch / worktree per issue.** Naming (from the live branch set):
  `feat/<slug>`, `fix/<slug>`, `research/<slug>`, `chore/<slug>`, `test/<slug>`,
  `task/<slug>`, `sync/<slug>`, `bug/<slug>`.
- Commit style: `area(#NN): summary` — e.g. `fix(config): rebalance reason/action token
  reservation (#92)`; merges recorded as `Merge #NN (PR #MM): …`.
- End commit messages with the required `Co-Authored-By` trailer.

## 4. PRs (tied to issues, orchestrator-reviewed)
- Open with `gh pr create --base <branch> --title "…" --body "…"`. Put `Closes #NN`
  (or `Fixes #NN`) in the body so the issue auto-closes on merge.
- The **main / orchestrator agent reviews every PR** before merge
  (`gh pr view`, `gh pr diff`, `gh pr review`).
- For `beta_mythos_fast` PRs, include the `DIVERGENCE-REVIEWED` token (see §6).

## 5. Orchestrator ↔ subagent contract
**Orchestrator (main agent):** files & chains issues; dispatches subagents; enforces stage
gating; reviews PRs; merges.
**Subagent:** works ONE issue on its tied branch; opens a PR that links the issue; reports
back for review.
**Every subagent dispatch MUST state:**
- the **issue number** it serves and the **stage** (analysis / fix / test);
- the **tied branch / worktree**, and that it must not switch branches;
- its **dependencies / what gates it**;
- the **deliverable** + acceptance criteria from the issue;
- "**open/link a PR with `Closes #NN`**; the orchestrator will review";
- **protected-file rules** (§6) and the commit / PR conventions above.

## 6. Branches & cross-branch sync (CRITICAL — read before ANY cross-branch change)
We run parallel long-lived branches, each a harness around a DIFFERENT model. **`beta` is
the hub:** shared harness features and fixes land on `beta` FIRST, then propagate **one-way,
outward, per-feature** — never a bulk merge, and **nothing ever merges back into `beta`.**

| Branch | Model / role | Sync rule |
|---|---|---|
| `main` | **Barebones** VibeThink harness | Curated one-way `beta → main`: only selected, stable, generic fixes. Most beta features deliberately do NOT flow to main. Never bulk-merge. |
| `beta` | **Feature-rich** VibeThink harness — the **hub** | Canonical line; all shared harness work starts here. Nothing merges INTO `beta` (not from `main`, not from any model branch). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NickalasLight/VibeHarness](https://github.com/NickalasLight/VibeHarness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
