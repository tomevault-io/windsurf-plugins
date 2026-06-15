---
trigger: always_on
description: This file instructs Claude Code (and the Claude GitHub App) how to behave as an automated manager for this repository.
---

# CLAUDE.md — Automated Repository Manager

This file instructs Claude Code (and the Claude GitHub App) how to behave as an automated manager for this repository.

---

## PR Automation

When triggered on a pull request:

**Step 1 — Resolve merge conflicts first (if any).**
Before reviewing code or checking CI, check whether the PR has merge conflicts. If it does, resolve them automatically as described in the Merge Conflict Resolution section below. Only proceed to steps 2–3 after the branch is clean.

**Step 2 — Check CI.**
Check whether **ALL** checks have passed — including tests and the Vercel deployment check.

**Step 3 — Review the code.**
Review for correctness, security issues, and adherence to the standards in this file.

**If ALL checks pass AND the code looks good** → merge the PR immediately using a squash merge. Leave a short comment explaining what was merged and why it passed.

**If ANY check failed OR the code has issues** → do NOT merge. Post a comment on the PR tagging the author (`@username`) with:
- A clear summary of what failed
- Specific steps they need to take to fix it
- A friendly but direct tone

Never merge a PR if any CI check or Vercel deployment check is still pending or has failed.

---

## Merge Conflict Resolution

When a PR has merge conflicts, resolve them automatically **before** doing anything else (before code review, before checking CI).

### Steps:

1. **Detect conflicts** — check the PR's mergeability via the GitHub API (`mergeable` field). If `mergeable: false`, proceed with resolution.

2. **Clone and resolve locally:**
   ```bash
   git clone https://github.com/magic-peach/reframe.git
   cd reframe
   git fetch origin
   git checkout <pr-branch>
   git merge origin/main
   # resolve conflicts (see rules below)
   git add .
   git commit -m "chore: resolve merge conflicts with main"
   git push origin <pr-branch>
   ```

3. **Conflict resolution rules — apply these in order:**

   - **`src/app/page.tsx`** — Always keep the `main` version exactly. This file has a known stable structure; contributor changes that add a `<main>` wrapper or duplicate JSX must be discarded in favour of main.
   - **`bun.lock` / `package-lock.json`** — Keep `bun.lock` from `main`. Delete any `package-lock.json` the PR introduced.
   - **`src/app/layout.tsx` metadata** — Keep all fields from `main`; merge in any *new* fields from the PR branch (e.g. new `openGraph` keys). If the PR duplicates an existing field, keep the `main` version of that field.
   - **CSS variables in `src/app/globals.css`** — Keep `main`'s variable names (`--bg`, `--surface`, `--border`, `--text`, `--muted`). Discard any renames from the PR branch.
   - **`.github/workflows/`** — Keep `main` versions of all workflow files. Only accept new workflow files from the PR if they don't conflict.
   - **All other files** — Use standard semantic merge: keep both sets of changes if they don't logically conflict. If two edits touch the same line and both are meaningful, apply the PR's change on top of `main`'s version (i.e., prefer preserving contributor intent where safe).

4. **Post a comment** on the PR after pushing the resolved branch, tagging the author:

   > Hey @username! This PR had merge conflicts with `main` — I've resolved them automatically and pushed the result. Here's what was changed:
   >
   > - **`src/app/page.tsx`**: kept the `main` version (your `<main>` wrapper was removed — this file has a fixed structure)
   > - **`bun.lock`**: regenerated from `main` (removed `package-lock.json`)
   > - _(list each file and what decision was made)_
   >
   > CI has been re-triggered. If anything looks wrong, feel free to push additional changes.

5. **Do not resolve conflicts that require domain judgement** — if two branches both meaningfully modify the same function body in `src/lib/ffmpeg.ts` or a complex component, do not guess. Instead post a comment asking the author to rebase manually and explain exactly which lines conflict.

---

## Auto-Labeling Rules

Apply labels automatically whenever a PR or issue is opened. Do not wait to be asked.

### On every new PR — apply labels immediately:

#### Difficulty (required — pick exactly one):

| Label | When to apply |
|---|---|
| `level:beginner` | Small/simple changes: typo fixes, minor copy edits, single-line tweaks, docs-only changes, adding a missing `aria-label` |
| `level:intermediate` | Moderate changes: new UI components, multi-file features, hooks, non-trivial bug fixes |
| `level:advanced` | Complex features: significant refactors, new lib integrations, multi-component architecture changes, FFmpeg filter changes |
| `level:critical` | Security fixes, critical bug patches, breaking changes that affect the entire app |

Use the **diff size and conceptual complexity** together — a 200-line diff that only reformats code is still `level:beginner`; a 20-line diff that changes FFmpeg filter logic is `level:advanced`.

#### Quality (optional — apply if clearly warranted):

| Label | When to apply |
|---|---|
| `quality:clean` | Code is well-structured, readable, consistent with project style, and has no obvious issues |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magic-peach/reframe](https://github.com/magic-peach/reframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
