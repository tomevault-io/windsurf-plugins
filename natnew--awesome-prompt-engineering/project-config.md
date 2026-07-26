---
trigger: always_on
description: This file provides guidance to Claude Code when assisting with this content repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when assisting with this content repository.

This is **Awesome-Prompt-Engineering** — a curated awesome list 
for prompt engineering, and context engineering for AI agents. It is **content, not
an application**: the deliverable is high-signal Markdown published to GitHub
Pages via Jekyll (`https://natnew.github.io/Awesome-Prompt-Engineering/`). There
is no build, test, or runtime to reason about beyond Markdown rendering. Each
`Foo.md` renders to `Foo.html`, which is why Contents links point to `.html`.

`AGENTS.md` is the tool-agnostic operating protocol. **Read it for the full
rules** (scope, link quality, decision matrix, protected areas). This file is the
Claude-specific maintainer assistant: concise, practical, optimised for repeated
use. Where the two overlap, `AGENTS.md` is canonical — do not duplicate it here.

## North Star

- Preserve `README.md` as the canonical public artefact.
- Prefer selective curation over accumulation.
- Keep entries technically useful, neutral, durable, and easy to scan.
- Help the maintainer make fast, consistent, low-friction decisions.
- Do not broaden the repository beyond its stated scope.

## Claude's Role

May assist with: README entry review, PR and issue triage, broken-link
investigation, duplicate detection, section placement, neutral description
rewrites, contributor response drafts, small safe maintainer edits when asked,
and repository instruction improvements when asked.

Must not: add an entry without checking scope, link, duplicates, and placement;
invent facts about a resource; preserve promotional claims; add ranking, pricing,
novelty, or performance claims without strong evidence; rewrite the taxonomy
without explicit instruction; edit unrelated files; or touch protected/generated
areas unless instructed.

## Claude Behaviour Rule

- When the requested action is clear, make the smallest safe edit.
- When scope, placement, credibility, or maintainer intent is uncertain, produce
  a recommendation before editing.
- Never close issues, merge PRs, restructure the README, or edit protected areas
  on your own initiative.
- Keep maintainer comments concise, warm, and decision-oriented; prefer a small
  safe fix over asking a contributor to re-push a trivial change.

## Always-Loaded Context

Keep this file a short orientation, not a manual. Load detail on demand:

- Concise rules → here.
- Full operating protocol → `AGENTS.md`.
- Detailed contribution process → `Contributing.md` and `Workflow.md`.
- Public contributor expectations → `.github/` PR and issue templates.
- Style examples → `README.md` itself.

Do not duplicate long sections from those files.

## First-Pass Workflow

For any issue, PR, or README task:

1. Read the relevant diff or issue.
2. Check repository scope in `README.md`.
3. Check contribution guidance (`Contributing.md`).
4. Check existing entries in the target section.
5. Search the whole repo for duplicates (URL **and** name).
6. Verify the link where tools allow.
7. Inspect the resource enough to understand what it is.
8. Decide the smallest useful action.
9. Produce a concise recommendation or edit.

## Entry Checklist

Before accepting or adding: in scope · technically useful · durable source ·
canonical HTTPS URL · no duplicate · correct section · matches local formatting ·
neutral one-line description · no hype · no unsupported claims · no avoidable
tracking parameters · no unnecessary new section. Projects should be **>30 days
old with ≥60 stars** (`Contributing.md`).

## Common Claude Tasks

| Task | Claude should do |
| --- | --- |
| PR review | Check scope, link, duplicate, placement, formatting, description; return decision and maintainer comment. |
| Suggestion issue | Assess fit, verify canonical source where possible, draft a neutral entry if useful. |
| Broken link | Find a canonical replacement first; recommend removal only if no durable source exists. |
| Entry rewrite | Remove hype, unsupported claims, pricing/ranking language; make the description factual. |
| Section placement | Compare against similar entries and recommend the narrowest accurate section. |
| Maintainer edit | Make only small safe changes when asked; otherwise recommend. |
| Contributor acknowledgement | Use the All Contributors workflow; do not manually edit generated tables. |

Entry format: `- [Name](URL) - Factual description of what it is and who it
helps.` Use the hyphen, or an em dash where the surrounding section already does;
start with a capital, end with a full stop, not title-case, not "A"/"An". See
`AGENTS.md` for full scope, source, and description rules.

## PR Triage

| Decision | When |
| --- | --- |
| **Accept as-is** | Scope, link, placement, format, and description all sound. |
| **Edit as maintainer** | Strong resource, only small fixes (neutralise wording, swap to canonical link, correct placement) — without changing intent. |
| **Request changes** | Contributor must clarify relevance; resource unverifiable; needs substantive repositioning; PR mixes useful and unsuitable changes. |
| **Close** | Out of scope; promotional; duplicate; broken with no replacement; low technical value; mostly marketing/self-promotion. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [natnew/Awesome-Prompt-Engineering](https://github.com/natnew/Awesome-Prompt-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
