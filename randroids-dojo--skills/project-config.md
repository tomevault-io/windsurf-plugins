---
trigger: always_on
description: Shared rules for every agentic coding tool working in {{PROJECT_NAME}}. Claude Code, Codex, Cursor, and any future agent: this file is mandatory reading before you write anything.
---

# AGENTS.md

Shared rules for every agentic coding tool working in {{PROJECT_NAME}}. Claude Code, Codex, Cursor, and any future agent: this file is mandatory reading before you write anything.

This repo uses the HTML-first variant of the spiral scaffold. The ledgers and contracts under `docs/` are `.html` files. This file (`AGENTS.md`) and `CLAUDE.md` stay as Markdown so Codex's root-down walk and Claude Code's project-memory import keep working.

Task tracking uses the HTML-backed Dots fork: the `dot-html` CLI (skill `task-tracking-dots-html`), which stores work items as `.html` files under `.dots/`. Do NOT use the Markdown `dot` CLI (skill `task-tracking-dots`) on this project, even if it is also installed. If only the Markdown variant is present, install the HTML fork first (`task-tracking-dots-html` ships an installer); do not silently fall back to Markdown dots.

Project pitch: {{PITCH}}

---

## RULE 1: NEVER USE EM-DASHES. EVER.

No em-dashes. Not in chat. Not in code comments. Not in commit messages. Not in PR descriptions. Not in docs. Not in test names. Not anywhere.

Use a period, comma, colon, parentheses, or rewrite the sentence. En-dashes are not substitutes. Plain hyphens are fine for ranges like `pages 10-20` and compound words.

Before every tool call that writes text, scan your output for Unicode codepoints U+2014 (em-dash) and U+2013 (en-dash). Rewrite if either is present.

If porting or quoting text from another source, strip all em-dashes from the ported text before committing.

---

## RULE 2: Read the GDD before making design decisions

The Game Design Document at `docs/gdd/` is the source of truth for what {{PROJECT_NAME}} is. Before proposing architecture, adding features, or changing data schemas, read it. If the GDD and your idea disagree, the GDD wins unless explicitly approved.

Before each implementation slice, read:

- `AGENTS.md`
- `README.md`
- `docs/IMPLEMENTATION_PLAN.html`
- `docs/WORKING_AGREEMENT.html`
- `docs/gdd/` (the relevant requirement files)
- `docs/PROGRESS_LOG.html`
- `docs/OPEN_QUESTIONS.html`
- `docs/FOLLOWUPS.html`
- `docs/GDD_COVERAGE.json`
- `docs/DEPENDENCY_LEDGER.html` (and run the Dependency Upgrade Gate from `docs/IMPLEMENTATION_PLAN.html`)
- `docs/PLAYTEST.html` and `docs/FUN_FACTOR_AUDIT.html` when coverage is >=80% done
- the current task backlog (HTML Dots via `dot-html`, stored under `.dots/`)

### Path-scoped Rules

Three additional rule files live under `.claude/rules/`. They are loaded automatically:

- **Claude Code** loads them based on the `paths:` glob in their frontmatter.
- **Codex** loads them via per-directory `AGENTS.md` symlinks (`docs/AGENTS.md`, `docs/gdd/AGENTS.md`) on its root-down walk.

The three rules:

- `.claude/rules/slice-discipline.md` (paths: source-code globs): no drive-by refactors, no speculative abstractions, refactor-in-slice.
- `.claude/rules/ledger-append-only.md` (paths: the four ledger files): never delete past entries.
- `.claude/rules/gdd-build-log.md` (paths: GDD section files): append a build log entry on every shipped feature.

When you add a source directory (`src/`, `app/`, `lib/`, `components/`, `pages/`, `tests/`, etc.) to this project, run this once to make slice-discipline visible to Codex inside that tree:

```
ln -sf ../.claude/rules/slice-discipline.md <src-dir>/AGENTS.md
```

Claude Code already picks up slice-discipline by path glob without the symlink.

---

## RULE 3: Stack constraints

{{STACK}}

Do not introduce new dependencies in core categories without explicit user approval.

---

## RULE 4: Commit messages and PR descriptions

- Write them as a human would.
- No AI attribution. No `Co-Authored-By: Claude`. No "Generated with Claude Code" footers. No mention of Claude, Anthropic, or AI assistance.
- Keep them short, clean, professional. Focus on the why, not the what.

---

## RULE 5: Autonomous PR loop

Operate continuously until the planned scope is complete. The loop definition lives in `docs/IMPLEMENTATION_PLAN.html`. The process contract lives in `docs/WORKING_AGREEMENT.html`. Follow both on every slice.

For every slice:

1. Read the rule, plan, product, progress, question, followup, coverage, dependency-ledger, and backlog documents listed in Rule 2.
2. Run the Dependency Upgrade Gate (see `docs/IMPLEMENTATION_PLAN.html`). If a watched dep is out of date, the upgrade IS the next slice unless red CI takes over.
3. Pick the highest-priority unblocked task from the implementation plan, dep ledger, GDD coverage gaps, followups, and active backlog.
4. Create one branch for one PR-sized slice. Never push directly to `main`.
5. Implement the slice fully using existing project patterns.
6. Add or update tests appropriate to the risk and surface area.
7. Update `docs/PROGRESS_LOG.html`, `docs/GDD_COVERAGE.json`, `docs/OPEN_QUESTIONS.html`, `docs/FOLLOWUPS.html`, `docs/DEPENDENCY_LEDGER.html`, and the GDD section when the work changes them.
8. Run the local verification suite. At minimum: dash checks, `git diff --check`, type-check, relevant unit tests, broader checks when warranted.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Randroids-Dojo/skills](https://github.com/Randroids-Dojo/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
