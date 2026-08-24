---
trigger: always_on
description: Subagents (launched via the Agent tool) start with zero context. Whenever launching a subagent, always copy the full contents of CLAUDE.md into the subagent prompt. This ensures rules like image downloading, frontmatter schemas, and content conventions are followed.
---

# Gitcoin 30 — Instructions for Claude

## Standing Rules

### Subagents must always receive CLAUDE.md context

Subagents (launched via the Agent tool) start with zero context. Whenever launching a subagent, always copy the full contents of CLAUDE.md into the subagent prompt. This ensures rules like image downloading, frontmatter schemas, and content conventions are followed.
### 1. Keep documentation in sync — always

Whenever frontmatter schemas, field names, valid enum values, or content conventions change, you **must** update **all four** in the same change:

1. **`CLAUDE.md`** — the authoritative schema reference (Frontmatter Schemas section below)
2. **`README.md`** — the public-facing Metadata / Frontmatter Fields section
3. **`.github/ISSUE_TEMPLATE/{type}.yml`** — the relevant issue template(s)
4. **`scripts/validate-content.ts`** — add/update the corresponding validation logic

Never update one without checking and updating the others.

### 2. Enum values live in `src/lib/types.ts` — single source of truth

`RESEARCH_TYPES` and `SENSEMAKING_CATEGORIES` are exported constants in `types.ts`. `parse-issue.ts`, `validate-content.ts`, and `content-validators.ts` import from there. Never duplicate these lists inline.

### 3. Clean code

- No dead code — remove hooks, props, or exports that nothing uses
- No duplication — if two places need the same constant, export it from one source
- Scripts stay consistent — publish scripts follow the same pattern: `parseCustomFields` captures data from the issue body, `addCustomFrontmatter` (async) processes and writes frontmatter fields including any file downloads

---

## Project Overview

A directory of public goods funding mechanisms, apps, campaigns, research, and case studies in the Ethereum ecosystem. Built with Next.js (App Router), React, Tailwind CSS v4, TypeScript.

## Content System

Markdown files live in `src/content/{apps,mechanisms,research,case-studies,campaigns}/`. Each file has YAML frontmatter followed by a markdown body.

### Directory Structure

```
src/content/
├── apps/           # Tools/platforms (Gitcoin Grants Stack, Juicebox, etc.)
├── mechanisms/     # Funding mechanisms (Quadratic Funding, Retroactive Funding, etc.)
├── research/       # Research articles and analysis
├── case-studies/   # Real-world case studies
└── campaigns/      # Funding rounds and campaigns
```

---

## Frontmatter Schemas

### Base fields (all content types)

```yaml
---
id: '1234567890'           # Unix timestamp string — use Date.now() when creating new files
slug: your-slug            # Kebab-case, must match the filename exactly
name: "Display Name"
shortDescription: "One-sentence description."
banner: /content-images/{category}/{slug}/banner.png   # Only if file exists — see Image Rules
tags:
  - tag-one
  - tag-two
lastUpdated: 'YYYY-MM-DD'
authors:                       # Always set when ingesting from an external source (see Ingesting section)
  - "Jane Smith"               # Names must exactly match entries in src/data/authors.json
  - "John Doe"                 # Add new authors to authors.json first if not already listed
relatedMechanisms:
  - quadratic-funding
relatedApps:
  - gitcoin-grants-stack
relatedCaseStudies:
  - some-case-study-slug
relatedResearch:
  - some-research-slug
relatedCampaigns:
  - some-campaign-slug
---
```

> `authors` is optional but recommended. Names must exactly match entries in `src/data/authors.json`. Add new authors to that file before referencing them. Missing `authors` produces a warning only and does not block CI. Unknown author names (not in `authors.json`) still produce an error and block CI.
>
> All five `related*` fields are parsed from frontmatter and rendered as related content sections on detail pages.

### Apps — additional fields

```yaml
logo: /content-images/apps/{slug}/logo.png   # Only if file exists — see Image Rules
featured: true                               # Team only — set by Gitcoin team. Surfaces in featured section on /apps and homepage
```

Logo images for apps must be the **white/negative (inverted) version** of the app's logo. They are displayed on dark backgrounds — a full-colour logo will not render correctly.

### Research — additional fields

```yaml
sensemakingFor: mechanisms   # Team only — set by Gitcoin team. Marks this as the sensemaking article for a category page
                             # Valid values: mechanisms | apps | campaigns | case-studies | research
                             # Only one article per category should have this set
                             # Use a wider 3:1 banner (e.g. 1800×600px) for sensemaking articles

researchType: Report         # Optional — MUST be one of: Book | Report | Opinion | Analysis | Perspective | Essay
                             # Validated by CI. Do not use other values.

ctaUrl: '/content-images/research/{slug}/book.pdf'  # Optional — URL for a CTA button on the detail page
                                                     # For PDFs in this repo: named book.pdf by convention, stored at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitcoinco/gitcoin_co_30](https://github.com/gitcoinco/gitcoin_co_30) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
