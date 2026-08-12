---
trigger: always_on
description: > Keywords: **MUST** / **NEVER** = mandatory. **SHOULD** = recommended unless there is a clear reason not to. **MAY** = optional.
---

# Global Claude Code Settings

> Keywords: **MUST** / **NEVER** = mandatory. **SHOULD** = recommended unless there is a clear reason not to. **MAY** = optional.

## Package Manager

- MUST use **yarn** (`yarn`, `yarn add`, `yarn remove`)
- NEVER use npm commands
- MUST use `yarn add` instead of manually editing package.json
- During upgrade work, if a dependency turns out to be unused, MUST propose removing it (`yarn remove`) instead of upgrading it

## General

- When today's date is needed, MUST run the `date` command to get it — NEVER rely on model's internal knowledge

## Git Operations

- NEVER perform git commit, push, or other git operations without explicit user permission
- MUST **check current branch** with `git branch` or `git status` before making changes
  - If the branch is different from expected, MUST ask the user which branch to use
- MUST **create a feature branch** before starting implementation work
  - MUST `git fetch` first and check `git log HEAD..origin/<default-branch>` — branch only from an up-to-date base. A local default branch dozens of commits behind produces work built on files the mainline has since moved, renamed, or re-linted, and the whole implementation then has to be relocated. "The repo looked fine when I read it" is not evidence — the working copy can be stale.
- MUST ask before running: `git commit`, `git push`, `git merge`, `git rebase`, etc.
- NEVER use `git add .` or `git add <directory>` — MUST add files individually
- NEVER delete untracked files
- NEVER push directly to main — MUST create a feature branch and open a PR
- MUST use merge commit (`--merge`) when merging PRs — NEVER use squash merge
- NEVER use `git rebase`
- NEVER use `git push --force` (unconditional overwrite). `git push --force-with-lease` is permitted only on a feature branch you just pushed yourself, after `git commit --amend` or similar local rewrite — it aborts safely if anyone else pushed in the meantime. NEVER force-push (any variant) to `main` or shared branches.
- MUST use commit message prefixes: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`
- When asked to 'create a PR' or 'PR、マージ', this MUST be interpreted as CREATE a pull request, not merge it
- MUST confirm the correct default/target branch before creating PRs
- Read-only operations (`git status`, `git diff`, `git log`) MAY be run freely
- SHOULD commit after each meaningful change (e.g., schema done → commit, utility functions done → commit)

## PR Bot Review Handling

After pushing to a PR, MUST triage **every** bot reviewer — not just the first one (CodeRabbit `coderabbitai[bot]`, Sourcery `sourcery-ai[bot]`, Codex, plus project-specific reviewers like Socket Security). Use the `/gh-review-loop` skill: it reads what all the GitHub-side bots posted on the latest commit (including inline threads that `gh pr view` omits), applies real fixes, pushes, and waits for re-review until every bot signs off, CI is green, and the user confirms.

Core principles it enforces — and which MUST hold even when triaging by hand:
- MUST NOT blindly apply suggestions — verify each against the actual codebase; bots disagree, so pick the right answer rather than satisfying both mechanically.
- Classify each comment: actionable fix (apply + add tests), valid nitpick (fix if cheap, else note as intentional), false positive / outdated (verify and skip with reason), rate-limited (note; re-check later).
- MUST commit fixes as `fix: address <bot-name> review comments` (name the specific bot), batched into one commit when possible.
- MUST post a follow-up PR comment summarizing what was addressed vs. deliberately skipped, so the human reviewer doesn't re-walk the bot threads.

## Change Scope Rules

- MUST only make changes that were explicitly requested — NEVER autonomously add features, tools, packages, or content
- MUST ask first if something additional seems needed
- MUST keep PR comments, commit messages, and documentation concise unless asked otherwise

## Bug Fix / Feature Request Workflow

When asked to fix a bug or implement a new feature:

1. MUST discuss and clarify requirements with the user
2. MUST create a GitHub issue summarizing the task once scope is clear
3. MUST create a plan file under the repository's `plans/` directory (e.g., `plans/fix-xxx.md` or `plans/feat-xxx.md`) — this file MUST be committed to the repo
4. MUST implement based on the plan
5. MUST include the user's original request in the PR description as a "User Prompt" section — when the request spans multiple conversation turns, consolidate the user's messages into a coherent summary that preserves all of the user's intent without omitting details. Clean up formatting but NEVER add content beyond what the user said. If there are multiple distinct requests, list each as a bullet point
6. MUST include detailed implementation approach, proposed steps, and key decisions in the PR description — important information discussed in chat MUST be persisted as files or PR comments, NEVER only in chat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isamu/claude](https://github.com/isamu/claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
