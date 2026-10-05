---
trigger: always_on
description: Read this first. It's the git + regen + test workflow that keeps agent edits safe and
---

# AGENTS.md — orientation for AI agents working on this repo

Read this first. It's the git + regen + test workflow that keeps agent edits safe and
reviewable. For the *quality bar* (what a good change looks like) read `CONTRIBUTING.md`.

---

## 1. Work in the checkout provided by the current environment

**Updated by Alaric's authorization, 2026-09-15:** Codex on Windows may read, edit,
regenerate, test, commit, and push directly from `C:\Users\alari\er-archipelago`.
This is a native Windows checkout, not the old Cowork filesystem mount. No Linux
sandbox, separate clone, or additional permission is required for routine work here.
Use the available native tools (PowerShell, git, Python, Cargo) and verify their
availability instead of applying historical sandbox limitations to this host.

- Inspect `git status`, the current branch, and remote state before editing. Preserve
  existing user changes; do not reset or overwrite them to make the tree clean.
- Work in one checkout for a change. Do not leave a draft in one working tree and
  push a different version from another. Use an isolated worktree when concurrency
  or conflicting local changes require it.
- Do not remove an index lock until you have established no live process owns it.
- If delegating work, identify the exact authorized checkout in each brief and
  re-verify load-bearing findings against that checkout.

The older Linux/Cowork recipes below apply only when actually running in that
sandbox. Their mount ban concerns `/sessions/*/mnt/er-archipelago` and similar
Cowork projections, **not this native Windows checkout**. In a Cowork session,
keep edits in the sandbox clone and transfer them through git; those mounts had
both stale-draft conflicts and observed truncated reads. Do not infer truncation
from ordinary native Windows reads or revert user files based on that old rule.

## 2. Which branch is live CHANGES — verify it, never trust this line

**`main` is the trunk on both repos.** But feature work does not always live there, and *this section
has been wrong twice*, in both directions:

- it once said "the active branch is `feat/matt-free-backbone-mvp`, NOT `main`" — by then that branch
  was 0 ahead of `main` and 36 behind, so following it checked you out onto a tree missing every
  recent commit;
- it was then corrected to a flat "`main` is the live branch, just clone and work on it" — which is
  what you are reading now, and it is **also** incomplete.

**Derived 2026-07-25 — a SNAPSHOT, not the answer. Re-run the commands below before you trust it:**

| repo | trunk | where live work is | note |
|---|---|---|---|
| `er-archipelago` (world) | `main` | **`main`** | `feat/natural-progression-mode` — named here as "live" on 07-24 — is now **0 ahead / 65 behind** `main`: merged and finished. No branch on origin holds live work (see below). Work on `main` |
| `from-software-archipelago-clients` (client) | `main` | `main` | push straight to `main`; that push is the Windows build gate (§4) |

Of the 26 non-`main` branches on the world origin, **21 are 0 ahead** (fully merged) and the other 5
carry 1-3 commits each while sitting 176-654 behind — stale scraps from July 8-21
(`agent/agents-md-client-note`, `agent/coverage-gate`, `agent/main-gate-grace-scadu-altus`,
`agent/surface-and-consumable`, `feat/spirit-ash-tiers`), not workplaces. Worth a skim before you
re-solve something, worth nothing as a base.

⚠️ **This table has now rotted three times, in three different directions:**
`feat/matt-free-backbone-mvp` (already dead when it was recommended) → a flat "`main` is the live
branch" (right trunk, wrong claim about where work happens) → `feat/natural-progression-mode` (true
the day it was written, merged two days later). Its half-life is about a week. **Derive, then read.**

So: **there is no standing answer to "which branch".** Do not read one out of this file. Derive it,
and if the repo state is ambiguous, ask Alaric — a wrong branch costs a whole session's work.

```bash
git ls-remote --heads origin | awk '{print $2}' | sort   # what actually exists, right now
# ahead/behind between trunk and a candidate branch (left = main-only, right = branch-only):
git fetch origin && git rev-list --left-right --count origin/main...origin/<branch>
```

> 🛑 **A `--depth N` clone makes `rev-list --left-right` LIE, silently.** The left-hand (main-only)
> number saturates at your clone depth, so on a `--depth 20` clone every branch reports exactly
> `20 <right>` and they all look equally divergent. It does not warn; it is a confident wrong number
> of precisely the kind CONTRIBUTING's "silent wrong answer" section is about. `git fetch --unshallow`
> before you measure, or don't shallow-clone at all. (Found the hard way, 2026-07-25.)

Read that output the way §7 wants you to read any derivation: a branch that is **0 ahead** of `main`
is a finished/merged branch and is not where work goes; a branch that is **behind** `main` needs a
rebase before you add to it. `origin/HEAD` may still point at a long-dead branch — ignore it.

(Rewritten 2026-07-24: the previous "`main` is the live branch" text was correct about the trunk and
wrong about where work happens, which is the same failure mode as the `feat/matt-free-backbone-mvp`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [4laric/er-archipelago](https://github.com/4laric/er-archipelago) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
