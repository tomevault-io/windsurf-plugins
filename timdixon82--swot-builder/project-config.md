---
trigger: always_on
description: Tim's writing voice is defined in `docs/writing-style.md`, and his visual brand in `docs/brand.md`. Both pages are in the global wiki.
---

# Claude Agent Team: Project Memory

Tim's writing voice is defined in `docs/writing-style.md`, and his visual brand in `docs/brand.md`. Both pages are in the global wiki.

This file is the shared memory for Tim Dixon's eight-agent Claude Code team. Every agent reads it. It records who Tim is, the standards the team must meet, how work is organised, who the agents are, the rules that may never be broken, and how the reference wiki works.

## Accessibility profile

Tim Dixon is severely sight-impaired. He uses VoiceOver on macOS and JAWS on Windows.

Every file the team produces, and every message sent to Tim, must be screen-reader-friendly:

- One H1 per file, then H2 and H3 headings in order, with no skipped levels.
- Plain language, at roughly Flesch-Kincaid grade 9 or below.
- Abbreviations expanded on first use.
- Descriptive link text, never "click here" and never a bare web address.
- No ASCII art, no decorative dividers, no emoji-led headings.
- Keyboard-only instructions. Never tell Tim to use a mouse or trackpad.
- Every visual element (image, chart, diagram, screenshot) described in full text.

## How questions are put to Tim

When the team needs decisions from Tim, Sonja presents them in a fixed, screen-reader-friendly format so Tim can answer quickly and unambiguously:

- Every question has a number from a single continuous sequence that never resets and never reuses a number. The sequence runs across the whole engagement, not per batch and not per session, so no two questions ever share a number. Each question is written with a "Q" prefix, for example Q24, and keeps that number for its whole life.
- Every answer option is lettered (A, B, C, and so on), with each option on its own line.
- Tim answers with the question number and the option letter together. For example, "Q24B" means option B for question Q24. He can answer several at once, such as "Q24B, Q25A, Q26C".
- A question with no fixed options (a free-text answer) still has a number, and Tim answers by quoting the number.
- Where Sonja has a recommendation, she names the recommended option so Tim can accept it in one step.
- Questions are always batched: an agent gathers all its open questions before sending them to Sonja, and Sonja puts the whole batch to Tim at once, never one at a time.

When the interactive picker is the right path, Tim opens `outputs/qbatch.html` (served by the dashboard server), navigates with arrow keys or number keys, copies the assembled answer string with the button at the bottom, and pastes it into the Claude Code session. The text-based Q-format remains the fallback for off-device sessions or when the dashboard server is not running.

## Compliance baseline

- **Accessibility:** the Web Content Accessibility Guidelines (WCAG) 2.2, at AAA conformance. Building to AAA satisfies the accessibility laws in scope: United Kingdom equality and public sector accessibility law, the European Accessibility Act, the Americans with Disabilities Act, and Section 508. The legal landscape is set out in `docs/accessibility.md`.
- **Data protection:** the United Kingdom General Data Protection Regulation (UK GDPR), for any personal data.
- **Security:** the OWASP Top 10, with each item mapped to a concrete defence.

The screen-reader-evidence gate is suspended until Carol can run automated screen-reader passes. The pattern and template remain at `docs/patterns/screen-reader-evidence.md` for when automation is available. Carol still runs all automated accessibility checks (axe-core, Pa11y, WCAG 2.2 AAA code review); the manual VoiceOver and JAWS evidence files are not required for release at this time.

## Work-folder convention

Each piece of work happens in its own folder under `.claude/work/<id>/`. A work folder holds a `brief.md` (the summary, requirements, routing plan, and the list of pre-approved GitHub actions) and a running `log.md`. The brief is the single source of truth for which GitHub actions may run without pausing to ask Tim. The templates and the slash command that scaffolds a work folder arrive in Stages 4 and 5.

No more than three work folders may be in `Status: active` at one time. The default is one. When the cap is hit, Sonja either parks an existing folder before opening a new one, or finishes one to merge before starting the next.

The `Status:` field takes exactly one of six lower-case values: `active`, `paused`, `parked`, `blocked`, `done`, `archived`. The glossary entry "Work folder status values" defines each, and `scripts/status-lint.sh` enforces the set in the pre-push hook and in continuous integration, so a brief with a malformed status fails the check before it reaches the main branch.

Before dispatching any agent that writes to the file system, Sonja checks the repository for iCloud sync-conflict artefacts (`<name> 2.<ext>`, `<name> 3.<ext>`, ...). If any are present, run `scripts/clean-icloud-duplicates.sh` first so agents do not pick up a stale duplicate.

## Inputs convention

The `Inputs/` folder is the drop zone for material Tim provides: source code, design briefs, screenshots, zip files, and any other assets. When an input arrives, Sonja:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timdixon82/SWOT-Builder](https://github.com/timdixon82/SWOT-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
