---
trigger: always_on
description: Author a rich, mobile-responsive HTML design doc for a code change, get the user's sign-off, merge the doc's predicted ask-gates (commands risky enough to confirm) into the permission config, isolate the work in a git worktree, execute the change step by step against the doc, then append a "Roadblocks & deviations" recap to the same HTML file. Trigger on requests for a design doc, technical spec, RFC, implementation plan, architecture sketch, or runbook before shipping a non-trivial change — eve
---


# Design and Ship

A three-phase workflow for non-trivial changes:

1. **Design** — author a self-contained HTML design doc with diagrams, pseudocode (not full source), the ask-gates (commands risky enough to confirm before running), and an agent runbook
2. **Ship** — after user sign-off, merge the ask-gates into `~/.claude/settings.json → permissions.ask`, enter a git worktree, execute the doc
3. **Recap** — append a "Roadblocks & deviations" section to the same doc, recording what actually happened

The reason for this shape: design docs that go stale during implementation are worse than no doc. By writing the recap into the same file, the doc stays accurate, the user gets a paper trail of decisions, and future work has a starting reference that matches reality.

---

## Phase 1 — Design

### Where the file lives

- Output path: `docs/<topic>-design-doc.html` inside the project repo (not the home folder, not `/tmp`)
- If the repo has no `docs/` directory, create one. Don't dump the file at the repo root
- Filename uses kebab-case and ends with `-design-doc.html` so it's grep-able

### Starting from the template

Read `assets/template.html` from this skill. It already contains the full CSS, the sticky-header, the layout grid, the sidebar `<aside class="outline">`, the inline `<nav class="toc">`, the Mermaid loader, the theme toggle, and the scrollspy. **Do not rewrite the CSS or scripts.** Replace only the marked placeholders:

- `__TITLE__` — page title (e.g., `Acme — Auth Design Doc`)
- `__SUBTITLE__` — short tagline (e.g., "Google OAuth + Email/Password")
- `__BRAND_LABEL__` — short label after the brand dot (e.g., "Auth Design")
- `__LEDE__` — one paragraph for the hero
- `__PILLS__` — `<span class="pill ...">…</span>` chips
- `__TOC_LIST__` — `<li>` entries for the inline TOC (mobile)
- `__OUTLINE_LINKS__` — `<a data-href="…">` entries for the sidebar (desktop)
- `__SECTIONS__` — the actual numbered sections
- `__THEME_STORAGE_KEY__` — distinct localStorage key (`dq-<topic>-theme`)
- `__FOOTER_NOTE__` — one-line footer

### Required sections (in this exact order)

Use the numbered `<span class="num">XX</span>` style for every section heading. Numbers run consecutively starting at 01.

| # | Heading | Must contain |
|---|---|---|
| 01 | Problem & root cause | Exact error string if there is one; one-paragraph root cause |
| 02 | Scope & non-goals | Two-column `.grid.cols-2` cards |
| 03 | Current architecture | One Mermaid `flowchart` of how things work today |
| 04 | Target architecture | One Mermaid `flowchart` of the desired state |
| 05+ | Affected flows | One `sequenceDiagram` per flow that materially changes |
| — | Files to edit | A table with columns: path / action tag / why |
| — | Code changes (shape, not source) | Before/after Mermaid + a per-file pseudocode table |
| — | Third-party dashboard steps | Only if external services (Google Cloud, Stripe, Supabase, etc.) need configuration |
| — | Environment variables | Table; only the vars that change |
| — | Test plan | **End-to-end only by default** (Cypress / Playwright). Strategy Mermaid + pseudocode of cases. See "Tests: e2e by default" below. |
| — | Commands that will ask first | Table + paste-ready JSON snippet for `permissions.ask` |
| — | Rollout & verification | Phased; acceptance checklist as `<ul class="checklist">` |
| — | Risk & rollback | Two-column `.grid.cols-2` cards |
| Last | Agent runbook | Preflight commands, ordered execution Mermaid, commands cheat-sheet table, definition-of-done `<ul class="checklist">`, must-nots list |

See `references/sections.md` for concrete examples of each.

### Tests: e2e by default

At scale, the test that actually catches regressions is the one that drives the app like a user does. Mocked unit tests pass in isolation and lie about integration. The design doc reflects this priority:

- **Always include an end-to-end test plan.** Cypress or Playwright, whatever the repo already uses. Pseudocode the cases; the strategy diagram explains how they assert (intercept, network probe, DOM assertion).
- **Do NOT add a unit-test plan, integration-test plan, or "expand existing mocks" task unless the user explicitly asks for one.** If existing unit tests would be broken by the change, the doc says "update broken unit tests" — not "add new ones".
- **Definition of done references only the e2e suite.** Don't make `npm run test` (unit) a gate unless the user asked for it.
- **Skip lint/build gates by default too** unless the project's pre-existing CI runs them — those are scaffolding, not signal.

If the user asks for unit tests on top of e2e ("also add unit tests for the new helper", "extend the vitest suite"), add the unit plan as a separate subsection (e.g., 13.x) under the existing Test plan section. Don't replace the e2e content.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nihadern/design-and-ship](https://github.com/nihadern/design-and-ship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
