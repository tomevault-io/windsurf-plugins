---
trigger: always_on
description: This file guides Claude Code when working in this repository.
---

# CLAUDE.md

This file guides Claude Code when working in this repository.

This repository is a public, maintained awesome list for generative AI, not an application codebase. There is no build, lint, or test workflow to run for normal review tasks. The `README.md` is the product.

Claude Code should read this file first, then use `AGENTS.md` as the shared repository operating protocol.

## North Star

* Preserve `README.md` as the canonical public artefact.
* Keep the list selective, durable, technically useful, neutral, and easy to scan.
* Help the maintainer make fast, consistent, low-friction decisions.
* Prefer small, precise edits over broad rewrites.
* Do not broaden the list beyond generative AI and clearly adjacent technical areas already represented in the README.

## Claude’s Role

Claude may assist with:

* PR review
* Issue triage
* README entry review
* Broken-link investigation
* Duplicate detection
* Section placement
* Neutral description rewrites
* Maintainer comment drafts
* Small safe maintainer edits when explicitly asked
* Improvements to agent instruction files when asked

Claude must not:

* Add entries without checking scope, link quality, duplicates, and placement
* Invent facts about a resource
* Preserve promotional claims
* Add ranking, pricing, novelty, adoption, or performance claims without strong evidence
* Rewrite the taxonomy without explicit instruction
* Edit unrelated files
* Touch protected areas unless instructed
* Ask contributors to make trivial fixes the maintainer can safely make

## Repository Facts

* `AGENTS.md` contains the full tool-agnostic operating protocol.
* `Contributing.md` contains contributor-facing rules.
* `Workflow.md` contains PR process detail.
* `.github/ISSUE_TEMPLATE/` contains public issue guidance.
* The `README.md` contains introductory content, Contents, Recently Added, infographics, and the main “Awesome Generative AI List”.
* The main list uses both bullet entries and tables. Match the surrounding section exactly.
* New entries usually go to the bottom of the relevant category unless local ordering clearly indicates otherwise.
* New categories should normally be handled separately.
* Protected areas include badges, Contents, Recently Added, banners, images, infographics, contributor blocks, generated sections, and licence text.

## Always-Loaded Context

Keep this file short. It is an orientation layer, not a manual.

Use this routing:

* Need general agent rules → read `AGENTS.md`
* Need contribution rules → read `Contributing.md`
* Need PR process → read `Workflow.md`
* Need style examples → inspect the target section in `README.md`
* Need contributor expectations → inspect `.github/ISSUE_TEMPLATE/`
* Need maintainer precedent → inspect recent issues and merged PRs where available

Do not duplicate long sections from those files here.

## First-Pass Workflow

For any PR, issue, or README task:

1. Read the user request.
2. Read the relevant issue, PR, diff, or target README section.
3. Check the repository scope.
4. Check `Contributing.md` if the task concerns a submission.
5. Check neighbouring entries for style and placement.
6. Search for duplicates.
7. Verify the link where tools allow.
8. Inspect the resource enough to understand what it is.
9. Choose the smallest useful action.
10. Produce a concise decision, edit, or maintainer comment.

## Entry Checklist

Before recommending acceptance or adding an entry, confirm:

* In scope
* Technically useful
* Credible source
* Canonical URL
* Durable link
* No duplicate
* Correct section
* Local format matched
* Neutral description
* No hype
* No unsupported claims
* No avoidable tracking parameters
* No unnecessary new section

## Source Preference

Prefer:

* Official repositories
* Official documentation
* Papers
* Technical reports
* Benchmarks
* Datasets
* Durable project pages
* Maintained tools and libraries
* High-quality reference material

Treat cautiously:

* Launch posts
* Vendor pages
* Thin wrappers
* Newsletter posts
* Social posts
* Unmaintained repositories
* Link farms
* Pages dominated by sales language
* Time-sensitive comparisons

## Description Rules

Default pattern:

`* [Name](URL) - Clear factual description.`

For tables, preserve the existing column structure.

Descriptions should:

* Start with a capital letter
* End with a full stop
* Be short and specific
* Avoid title case
* Avoid starting with “A” or “An”
* Avoid marketing taglines
* Explain what the resource is, not why it is exciting

Remove or neutralise:

* “best”
* “latest”
* “most advanced”
* “powerful”
* “revolutionary”
* “cutting-edge”
* “game-changing”
* “industry-leading”
* “fastest”
* Unsupported performance, adoption, maturity, or pricing claims

## Section Placement

| Situation                             | Action                                                |
| ------------------------------------- | ----------------------------------------------------- |
| Exact fit in an existing section      | Place there.                                          |
| Fits two sections                     | Choose the more specific or more discoverable one.    |
| Similar to neighbouring entries       | Place near those entries if local ordering allows.    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [natnew/Awesome-Generative-AI](https://github.com/natnew/Awesome-Generative-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
