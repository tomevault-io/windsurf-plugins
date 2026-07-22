---
trigger: always_on
description: Agent-facing guide for working in the dbt documentation repository.
---

# AGENTS.md — docs.getdbt.com

Agent-facing guide for working in the dbt documentation repository.

## Project overview

- **Framework**: Docusaurus 3.7.0, deployed on Vercel
- **PR base branch**: `current`
- **Content format**: Markdown (`.md`) and MDX (`.mdx`)

### Key directories

```
website/
  docs/           # Product documentation
  blog/           # Developer blog posts
  snippets/       # Reusable partials and snippets
  src/components/ # React components available in MDX
  src/theme/      # Docusaurus theme overrides
  static/img/     # Images and icons
  plugins/        # Custom Docusaurus plugins
```

### Key config files

| File | Purpose |
|------|---------|
| `website/docusaurus.config.js` | Site config, navbar, footer, plugins |
| `website/sidebars.js` | Left sidebar navigation structure |
| `website/dbt-versions.js` | Version definitions, versioned pages/categories |
| `website/constants.js` | Global terminology constants |
| `website/blog/authors.yml` | Blog post author profiles |
| `website/blog/categories.yml` | Blog tag/category definitions |
| `website/vercel.json` | Contains page redirects |
## Dev commands

```bash
cd website
npm start          # Local dev server (port 3000)
npm i              #  Install dependencies
npm run build      # Production build
npm test           # Jest tests (watch mode)
npm run lint       # ESLint with cache
npm run lintAll    # ESLint auto-fix
```

## Branding & terminology

These rules are **mandatory** — incorrect branding is the most common docs error.

### Product names (all case-sensitive)

| Name | Usage | Notes |
|------|-------|-------|
| **dbt** | Always lowercase | Generic references across all products |
| **dbt Core** | Lowercase `dbt`, capital `C` | dbt versions ≤ 1.x |
| **dbt Fusion engine** | Or just "Fusion" in docs | dbt versions ≥ 2.x |
| **dbt platform** | Not "dbt Cloud" | The cloud-based platform (formerly dbt Cloud) |
| **dbt Labs** | The company | When docs say "we," this is who "we" is |

### Feature proper nouns (capitalize these)

Studio IDE, Canvas, Insights, Catalog, Mesh, Orchestrator, Semantic Layer, Copilot

All other features (models, environments, configs, settings) are common nouns.

### Use `<Constant>` for product names when available

```jsx
<Constant name="dbt_platform" />   // → "dbt platform"
<Constant name="fusion_engine" />  // → "dbt Fusion engine"
<Constant name="studio_ide" />      // → "Studio IDE"
```

Refer to `website/constants.js` for constant names

### Third-party brands

Respect their branding. Use official names (VS Code, not VScode; Microsoft Entra ID, not Azure AD).

### Acronyms

All caps except `dbt`: YAML, SQL, JSON, IDE. Spell out on first use.

## Writing style

- **Voice**: Active, second person ("you"), conversational
- **Commas**: Oxford comma required
- **Titles**: Sentence case (not Title Case)
- **Spelling**: US English (standardize, not standardise)
- **Emphasis**: Italics for emphasis, bold for UI elements only
- **Code font**: Filenames, commands, params, directory paths, branch names
- **Placeholder text**: `SCREAMING_SNAKE_CASE` (no brackets and no angular brackets)
- **Avoid**: Latin abbreviations (i.e., e.g., etc.) — use "that is," "for example," "and more"
- **UI elements**: Bold the element name — "Click **Submit**" (not "Click the **Submit** button")
- **Links**: Never "Click here" — use descriptive text: "refer to the [dbt Labs doc site](https://docs.getdbt.com/)"

Full guide: `contributing/content-style-guide.md`

## Content types & title conventions

| Type | Title pattern | Example |
|------|---------------|---------|
| Conceptual | "About [noun]" | "About incremental models" |
| Referential | Descriptive noun phrase | "Supported data platforms" |
| Procedural | Gerund (verb + -ing) | "Setting up continuous integration" |
| Guide | Gerund, general scope | "Managing repository settings" |
| Quickstart | Gerund, no "quickstart" in title | "Getting started with Snowflake" |
| Cookbook | "How to [verb] [topic]" | "How to calculate ARR using metrics" |

Full guide: `contributing/content-types.md`

## Frontmatter

### Docs pages

```yaml
---
title: "Page title"
id: "unique-identifier"
description: "SEO description"
sidebar_label: "Sidebar text"
pagination_next: "docs/path/to/next-page"
pagination_prev: null
hide_table_of_contents: false
---
```
Do not use constants in frontmatter.

### Blog posts

Filename: `website/blog/YYYY-MM-DD-slug.md`

```yaml
---
title: "Post title"
description: "Compelling sentence from the article"
slug: seo-optimized-slug
authors: [author_slug]
tags: [appropriate-tag]
hide_table_of_contents: false
date: YYYY-MM-DD
is_featured: true
---
```

Insert `<!-- truncate -->` after intro paragraphs (before first `##`).

Authors: `website/blog/authors.yml` · Tags: `website/blog/categories.yml`

## Versioning

### Version definitions

Managed in `website/dbt-versions.js`. First entry = latest/default.

| Version | Display | Status |
|---------|---------|--------|
| 2.0 | dbt Fusion engine (Latest) | Prerelease |
| 1.12 | dbt platform (Latest Core) | Stable |
| 1.11 | Core v1.11 | EOL 2026-12-18 |
| 1.10 | Core v1.10 (Compatible/Extended) | EOL 2026-06-15 |

### Version content blocks

Show/hide content based on selected version:

```jsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbt-labs/docs.getdbt.com](https://github.com/dbt-labs/docs.getdbt.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
