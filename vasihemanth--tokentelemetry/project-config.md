---
trigger: always_on
description: **Every push to a branch destined for `main` that contains at least one
---

# TokenTelemetry — Claude Code project rules

## ⚠️ Pre-push policy (enforced by hook)

**Every push to a branch destined for `main` that contains at least one
`feat:` commit must update `UPDATE.json` at the repo root.**

This is enforced by `.claude/hooks/enforce-update-json.py`, wired up via
`.claude/settings.json` as a `PreToolUse` hook on the `Bash` tool. The hook:

1. Detects `git push` / `gh pr create` invocations (via proper shell
   tokenisation — not substring matching, so `echo "git push"` is fine).
2. Skips enforcement if we're on `main` itself.
3. Scans `git log origin/main..HEAD` for Conventional Commit subjects
   matching `feat:` / `feat(scope):` / `feat!:` etc.
4. If no `feat:` commits are present (only `fix:`, `chore:`, `docs:`,
   `refactor:`, `style:`, `test:`, `ci:`) — **allows the push silently**,
   because the change isn't user-facing in a way the banner should announce.
5. If at least one `feat:` commit IS present, requires `UPDATE.json` to
   appear in the branch's diff vs `origin/main`. Otherwise: deny.

**Why this rule, not "block every push":** UPDATE.json is a CURATED feed.
Forcing every fix/chore/docs push to add a fake entry would pollute it with
noise users learn to ignore. The banner is only valuable when it announces
something users actually want to know about — i.e., features.

**Why this rule, not "trust the maintainer":** for actual `feat:` work it's
easy to forget the file, and a stale banner is worse than no banner
(eroded trust, users on prior versions miss the update prompt entirely).

## UPDATE.json schema

```json
{
  "releases": [
    {
      "tag": "YYYY-MM-DD",
      "title": "Short release headline (≤50 chars)",
      "highlights": [
        {
          "title": "One-line feature name",
          "description": "1–2 sentences: what changed and why a user should care. No marketing fluff; concrete benefits.",
          "href": "/settings"
        },
        {
          "title": "Another feature",
          "description": "..."
        }
      ]
    },
    {
      "tag": "(prior releases stay below — drawer shows up to 6 newest)"
    }
  ],
  "release_url": "https://github.com/VasiHemanth/tokentelemetry/commits/main"
}
```

### Field rules
- **`tag`**: ISO date (`YYYY-MM-DD`). Used as a fallback heading when `title` is absent.
- **`title`**: short headline. Shows as the section heading in the drawer.
- **`highlights`**: 1–5 bullets per release. Anything past 5 is truncated.
- **`href`** (optional): internal route (`/settings`) renders as `<Link>` keeping nav in-app; external URL (`https://…`) opens a new tab.

### Each release entry should
- Go to the **top** of `releases[]` (newest first — drawer renders top-down)
- Have a description that helps a non-technical user understand *why* this matters, not just *what* changed
- Use an `href` when a single page is the natural landing spot for the change (e.g. a new feature page, a redesigned section)
- Credit external contributors for major community-raised bugs/fixes: end the
  relevant highlight's description with "Thanks to <github-username> for
  reporting and fixing this (PR #N)." Reserve credit for significant external
  contributions so it stays meaningful; maintainer changes carry no credit line.

## When the hook gets in your way

The hook is intentionally narrow. Cases:

1. **Pure `fix:`/`chore:`/`docs:`/etc. branch**: hook allows silently. You don't need to touch UPDATE.json.
2. **Mixed branch with one `feat:` + several `fix:` commits**: hook still requires UPDATE.json — write an entry for the feature, fixes get a free ride.
3. **`feat:` branch where the feature genuinely isn't user-visible** (e.g. internal refactor mis-labeled `feat:`): re-label the commit to `refactor:` / `chore:` and amend, OR add a UPDATE.json note explaining what you shipped to users.
4. **Pushing main itself** (rebase, force-push for cleanup): hook skips automatically.
5. **Repo without `origin/main`**: hook allows through (assumes you know your setup).
6. **Last resort**: `claude --no-hooks` for that session — but if you find yourself reaching for this often, the rule is mis-tuned and worth revisiting.

## Pre-push merge validation (two gates)

After issue #91 (vulnerable + unused deps merged un-reviewed during the
fast remote-access ship), every change destined for `main` passes two gates:

1. **`.claude/hooks/prepush-claude-review.py`** (local, PreToolUse on `Bash`).
   On a `git push` / `gh pr create` from a non-main branch, it diffs
   `origin/main..HEAD`, sends the diff to your local `claude` CLI for a focused
   review (dependency hygiene, remote-exposure / auth-bypass regressions,
   committed secrets, injection-class bugs), and **denies the push only on an
   explicit high-confidence `block` verdict**. It **fails open**: missing
   `claude` CLI, empty/huge diff, timeout, or unparseable output all ALLOW the
   push — a flaky reviewer never blocks legit work. Reuses the push-detection
   helpers from `enforce-update-json.py` (imported, not duplicated). Skips
   docs/asset-only pushes. Bypass with `--no-hooks`.
2. **`.github/workflows/security-audit.yml`** (CI, deterministic). Runs
   `npm audit --omit=dev --audit-level=high` across root / `frontend` / `website`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VasiHemanth/tokentelemetry](https://github.com/VasiHemanth/tokentelemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
