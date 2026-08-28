---
trigger: always_on
description: Guidance for Claude Code working in this repository.
---

# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this repo is

A curated knowledge base of web security learning resources. Data lives in
`data/categories.yml` and `data/entries/*.yml`; the three language READMEs
(`README.md`, `README-zh.md`, `README-jp.md`) and `data/index.json` are
**generated** from that data by `scripts/generate.py`. The same data backs a
Claude Skill at `skills/awesome-web-security/` so AI agents can query the
list at runtime.

No build, no test suite, no application runtime. Every change is either
YAML data, Python tooling, or Markdown docs.

## Files of note

- `data/categories.yml` — section tree, anchors, ToC. Edit carefully.
- `data/entries/*.yml` — one file per section. Source of truth for entries.
- `data/templates/preamble.md` / `postamble.md` — verbatim header/footer in
  the generated READMEs.
- `scripts/generate.py` — YAML → README.md / README-zh.md / README-jp.md /
  data/index.json.
- `scripts/migrate.py` — one-shot importer that originally produced the YAML
  from the old hand-edited READMEs. Kept for reference; do not re-run on a
  populated `data/`.
- `scripts/verify_schema.py` — entry schema validation; CI gate.
- `scripts/verify_anchors.py` — ensures no anchor used by external links is
  removed; CI gate.
- `scripts/verify_skill.sh` — lints `marketplace.json` and `SKILL.md`.
- `scripts/ci/pr_review.py` — auto-review bot. Default LLM is
  `openai/gpt-4.1-mini` via GitHub Models; override via the `REVIEW_MODEL`
  env var in the workflow. Contributor language is detected
  deterministically (Han / Kana regex on PR body) and passed to the LLM
  as a directive; the LLM never owns language detection.
- `scripts/ci/templates/comment.{en,zh,jp}.md` — localized review comments.
- `.github/workflows/pr-review.yml` — runs the bot on every PR.
- `.github/workflows/pr-review-backlog.yml` — dry-run over open PRs (manual,
  workflow_dispatch). Stashes trusted scripts to `/tmp/awsec-trusted/` before
  iterating PR refs, so `pr_review.py` always runs from the base branch even
  when an attacker-controlled PR is checked out.
- `.github/workflows/health-link-check.yml` — daily lychee scan over the
  generated READMEs and `data/index.json`; surfaces broken links via a single
  rolling issue labelled `health/link-check`. Replaces the previous
  `validate.yml`.
- `.github/workflows/post-merge-archive.yml` — after merges that touch
  `data/entries/**`, submits eligible entries (active, no `archive_url` yet,
  not opted out) to Wayback Machine and commits the resulting `archive_url`
  back to the YAML with `[skip ci]`. Driven by `scripts/ci/archive.py`.
- `.github/ISSUE_TEMPLATE/propose-resource.yml` — GitHub Form for proposals.
- `.github/PULL_REQUEST_TEMPLATE.md` — self-checklist mirroring RUBRIC.md.
- `.claude-plugin/marketplace.json` — declares the Claude plugin / Skill
  package distributed via this repo.
- `skills/awesome-web-security/SKILL.md` — the Skill body. Lives here so the
  same repo serves humans and AI agents.
- `RUBRIC.md` — the five-dimension scoring rubric the bot applies; also the
  contributor self-check reference.
- `CONTRIBUTING.md` — contributor flow. Treat as authoritative for PR
  procedure; do not duplicate its contents here.

## Editing rules

- **Never edit `README.md`, `README-zh.md`, or `README-jp.md` directly.**
  They are regenerated from `data/` by `scripts/generate.py`. Edits will be
  blown away on the next regeneration.
- To add or modify an entry, edit `data/entries/<category>.yml`.
- To add a new category, edit `data/categories.yml` first, then add a new
  `data/entries/<category-key>.yml`. Run `scripts/verify_anchors.py`
  afterwards to confirm no external-facing anchor was lost.
- After any data change, run `python3 scripts/generate.py` to refresh the
  generated files locally before committing. CI will reject divergence.

## Entry schema

```yaml
- id: xss-google-app-security                   # kebab-case, unique
  url: https://www.google.com/...               # required, https preferred
  title: Cross-Site Scripting – Application Security – Google
  author:
    name: Google                                # required if author exists
    url: https://www.google.com/                # optional
  category: xss                                 # must exist in categories.yml
  type: article                                 # article|tool|cheatsheet|video|book|community|payload-list
  languages: [en]                               # subset of: en|zh|jp|tr|universal
  difficulty: intro                             # intro|intermediate|advanced
  date_added: 2017-01-29                        # ISO date
  archive_url: null                             # filled by post-merge-archive workflow
  last_checked: null                            # filled by weekly-health workflow
  fingerprint: null                             # content fingerprint, filled by health
  status: active                                # active|dead|archived-only|quarantined
```

Optional fields:

- `raw_rest` is the README description text that follows the title link
  (the generator emits `- [Title](url) - {raw_rest}.`). It is **public-facing,
  third-party-readable** content; treat it like user-facing copy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christophercruz98112/awesome-web-security](https://github.com/christophercruz98112/awesome-web-security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
