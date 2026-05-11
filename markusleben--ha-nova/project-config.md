---
trigger: always_on
description: Version: 0.25 (2026-02-01)
---

# AGENTS.md

Version: 0.25 (2026-02-01)

Start: say hi + 1 motivating line.
Work style: Be radically precise. No fluff. Pure information only (drop grammar; min tokens).

## Project
- GitHub User: see `.env` <GH_USER>

## Agent Protocol
- Contact: Markus Leben (@markusleben).
- “Make a note” => edit AGENTS.md (Ignore `CLAUDE.md`, symlink for AGENTS.md).
- Editor: `cursor <path>`.
- New deps: quick health check (recent releases/commits, adoption).
- When asked to update the `AGENTS.md` to the latest version:
  1. Fetch `https://raw.githubusercontent.com/markusleben/agents.md/main/AGENTS.md`.
  2. Check if a newer version exists and merge without losing local changes.

## Skill Files — English Only [DON'T SKIP – IMPORTANT]
- ALL skill files (`skills/**/*.md`, agent templates, reference docs) MUST be 100% English.
- No German text — not in examples, not in dispatch tables, not in comments, not in inline strings.
- The only exception is proper nouns (attribution names, entity IDs like `light.wohnzimmer` from a real HA instance).
- Output localization (translating headings/labels for the user) happens at runtime per the context skill — never baked into skill source files.

## Code Quality [DON'T SKIP – IMPORTANT]
- All generated code must be self-reviewed before being presented.
- Continue reviewing and fixing until no further issues are found.
- Do not show partial or unreviewed code to the user.

## Guardrails
- ALWAYS review the written code, don't present it to the user without! Review it so long, until the review won't find any more issues
- Use `trash` for deletes.
- Use `mv` / `cp` to move and copy files.
- Bugs: add regression test when it fits.
- Keep files <~400 LOC; split/refactor as needed.
- Simplicity first: handle only important cases; no enterprise over-engineering.
- New functionality: small OR absolutely necessary.
- NEVER delete files, folders, or other data unless explicitly approved or part of a plan.
- Before writing code, strictly follow the research rules below.

## Research
- Always create a spec, even if minimal
- Prefer skills if available over research
- Prefer researched knowledge over existing knowledge when skills are unavailable
- Research: Exa to websearch early, and Ref to seek specific documention or web fetch.
- Best results: Quote exact errors; prefer 2025-2026 sources.

## Git
- Always use `gh` to communicate with GitHub.
- **Multi-Account:** Before remote ops (push, repo create, PR), run `gh auth status`. If active account ≠ `Project.GitHub Account`, ask user before proceeding.
- Use `gh auth switch --user <GitHub User>` to switch between GitHub accounts.
- GitHub CLI for PRs/CI/releases. Given issue/PR URL (or `/pull/5`): use `gh`, not web search.
- Examples: `gh issue view <url> --comments -R owner/repo`, `gh pr view <url> --comments --files -R owner/repo`.
- Conventional branches (`feat|fix|refactor|build|ci|chore|docs|style|perf|test`).
- Safe by default: `git status/diff/log`. Push only when user asks.
- `git checkout` ok for PR review / explicit request.
- Branch changes require user consent.
- Destructive ops forbidden unless explicit (`reset --hard`, `clean`, `restore`, `rm`, …).
- No repo-wide S/R scripts; keep edits small/reviewable.
- Avoid manual `git stash`; if Git auto-stashes during pull/rebase, that’s fine (hint, not hard guardrail).
- If user types a command (“pull and push”), that’s consent for that command.
- Big review: `git --no-pager diff --color=never`.
- **Review clearance is commit-specific:** any new relevant delta after the last bot-reviewed commit invalidates prior review clearance.
- **Relevant delta means:** any code, tests, docs-that-change-behavior, scripts, workflow files, release metadata, installer/update flow, or release notes change that alters the commit to merge or tag.
- **Push is not review:** a new push never inherits the previous clean review state.
- **A clean bot result is SHA-specific:** it applies only to the exact commit SHA it reviewed; any later SHA is unreviewed until the full cycle completes again.
- **Codex advisory rule:** `codex-review-gate` is advisory on `main`; do not treat it as a required branch-protection gate for routine PRs.
- **No local-only release shortcuts:** if a follow-up fix matters enough to keep, it must go through GitHub review before merge/tag/release.
- **Release/tag/publish gate:** never create/move a release tag, start RC/final publish, or call a commit release-ready unless the exact remote commit state intended for tag/release is represented by the latest fully reviewed PR state with no unreviewed deltas beyond it.
- **Release-bound review hardening:** do local/self review before opening the PR. After the PR exists, use the fast path only: after each relevant fix, run targeted local verification, push immediately, and trigger `@codex` immediately. After PR creation, Codex bot + CI are the review path; do not add extra local review gates in between.
- **Manifest-label rule:** if a PR changes `package.json`, `package-lock.json`, `nova/package.json`, or `nova/package-lock.json`, add `manifest-review:approved` immediately after `gh pr create` and before `@codex` / `gh pr checks --watch`.
- **PR Merge / Release Commit Gate — MANDATORY CHECKLIST (do NOT skip any step):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markusleben/ha-nova](https://github.com/markusleben/ha-nova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
