---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repository is a public, maintained awesome list for Physical AI and embodied AI, not an application codebase. The `README.md` is the product. There is no application build or unit-test suite; the only checks are content checks (link checking, markdown lint, entry counts) and the standalone documentation site under `website/`.

Claude Code should read this file first, then use `AGENTS.md` as the shared repository operating protocol.

## North Star

* Preserve `README.md` as the canonical public artefact.
* Keep the list selective, durable, technically useful, neutral, and easy to scan.
* Help the maintainer make fast, consistent, low-friction decisions.
* Prefer small, precise edits over broad rewrites.
* Do not broaden the list beyond Physical AI, embodied AI, robotics, and clearly adjacent technical areas already represented in the README.

## Claude's Role

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
* Move or add a second **Start here** marker without instruction
* Edit `README.md` and the matching `website/docs/` page out of sync
* Edit unrelated files
* Touch protected areas unless instructed
* Ask contributors to make trivial fixes the maintainer can safely make

## Commands

There is no application build. The repository runs content checks; CI mirrors these in `.github/workflows/`.

* Entry-count band — every canonical category must hold 15–25 entries:
  `python scripts/check_entry_counts.py`
* Markdown lint (advisory, matches the `lint` workflow):
  `npx --yes -p remark-cli -p remark-preset-lint-recommended -p remark-mdx remark --use remark-mdx --use remark-preset-lint-recommended --quiet --frail README.md "website/docs/**/*.md" "website/docs/**/*.mdx"`
* Link check (matches the `link-check` workflow; requires the `lychee` binary):
  `lychee --no-progress --max-retries 2 README.md "website/docs/**/*.md" "website/docs/**/*.mdx"`
  Allowed exceptions live in `.lycheeignore`.
* Documentation site (Docusaurus, Node ≥ 18, run from `website/`):
  `cd website && npm install && npm start` to preview, `npm run build` to build.

## Repository Facts

* `AGENTS.md` contains the full tool-agnostic operating protocol.
* `CONTRIBUTING.md` contains contributor-facing rules, the per-type inclusion gates, formatting conventions, the tag scheme, and the **Start here** update process.
* `website/docs/` is a standalone Docusaurus site. `website/docs/categories/<slug>.mdx` mirrors each README category and must stay in sync with it. `website/docs/workflow.mdx` and `workflow-review.mdx` document the PR and curation processes.
* `.github/ISSUE_TEMPLATE/` contains the public issue forms (new resource, remove resource, category proposal, curation review).
* The `README.md` contains a Get Started block, Contents, the canonical categories, and an Appendices block.
* The main list uses bullet entries with an em dash separator: `- [Name](URL) — Description.` Many entries carry a `<!-- tags: ... -->` comment on the next line. Match the surrounding section exactly.
* Each category has exactly one **Start here** entry, updated deliberately and changed in both `README.md` and the matching `website/docs/categories/<slug>.mdx`.
* New entries usually go to the bottom of the relevant category unless local ordering clearly indicates otherwise.
* New categories require a Category proposal issue with at least three seed entries and are handled separately.
* The list is reviewed monthly via a scheduled curation-review issue; `REVIEW.md` (local only) holds the full process.
* Protected areas include badges, the Get Started block, Contents, banners, images under `assets/`, **Start here** markers, contributor blocks, generated sections, and licence text.

## Always-Loaded Context

Keep this file short. It is an orientation layer, not a manual.

Use this routing:

* Need general agent rules → read `AGENTS.md`
* Need contribution rules → read `CONTRIBUTING.md`
* Need PR or curation process → read `website/docs/workflow.mdx` and `workflow-review.mdx`
* Need style examples → inspect the target section in `README.md`
* Need contributor expectations → inspect `.github/ISSUE_TEMPLATE/`
* Need maintainer precedent → inspect recent issues and merged PRs where available

Do not duplicate long sections from those files here.

## First-Pass Workflow

For any PR, issue, or README task:

1. Read the user request.
2. Read the relevant issue, PR, diff, or target README section.
3. Check the repository scope.
4. Check `CONTRIBUTING.md` if the task concerns a submission.
5. Check neighbouring entries for style and placement.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [natnew/awesome-physical-ai](https://github.com/natnew/awesome-physical-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
