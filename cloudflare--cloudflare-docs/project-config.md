---
trigger: always_on
description: This file helps AI agents understand the structure, tooling, and conventions of the `cloudflare-docs` repository so they can make correct, buildable changes.
---

# AGENTS.md — Cloudflare Docs

This file helps AI agents understand the structure, tooling, and conventions of the `cloudflare-docs` repository so they can make correct, buildable changes.

## Repository overview

This is the source for [developers.cloudflare.com](https://developers.cloudflare.com). It is an **Astro** site using the **Starlight** documentation framework. Content is authored in **MDX** (Markdown + JSX). The site is deployed as a Cloudflare Worker.

- **Node.js**: 24.x
- **Package manager**: pnpm (use `pnpm install --frozen-lockfile` to install)
- **Primary branch**: `production` (not `main`)

## Directory structure

```
cloudflare-docs/
├── src/
│   ├── content/
│   │   ├── docs/           # 5,400+ MDX pages — the user-facing documentation
│   │   ├── partials/       # 1,200+ reusable MDX snippets (by product)
│   │   ├── changelog/      # Product changelogs (by product subdirectory)
│   │   ├── glossary/       # Glossary term definitions (YAML)
│   │   ├── products/       # Product metadata (YAML, 135 files)
│   │   └── ...             # Other data collections (plans, fields, models, etc.)
│   ├── components/         # Custom Astro + React components
│   │   ├── index.ts        # Central re-export barrel — all MDX imports come from here
│   │   └── overrides/      # Starlight component overrides (Banner, Footer, Head, etc.)
│   ├── schemas/            # Zod schemas for all content collections
│   ├── plugins/            # Remark, Rehype, Starlight, and Expressive Code plugins
│   ├── icons/              # Product SVG icons (~110)
│   ├── assets/             # Processed images (optimized by Astro)
│   ├── styles/             # CSS (Tailwind 4)
│   ├── pages/              # Dynamic route pages (changelog, glossary, search)
│   └── util/               # Shared utility functions
├── public/                 # Static files served as-is (images, redirects, robots.txt)
├── worker/                 # Cloudflare Worker for serving the site
├── bin/                    # Build scripts and CI helpers
│   └── fetch-skills.ts     # Downloads skills.tar.gz from middlecache, extracts to skills/
├── skills/                 # Agent Skills served at /.well-known/skills/ — GENERATED, do not edit
│                           # Fetched from https://middlecache.ced.cloudflare.com/v1/cloudflare-skills/skills.tar.gz
│                           # by bin/fetch-skills.ts, which runs automatically via prebuild/predev hooks.
│                           # skills/ is in .gitignore and is NOT committed to the repository.
├── .flue/                  # Flue cloudflare-docs-bot — see .flue/AGENTS.md
├── astro.config.ts         # Astro + Starlight configuration
├── ec.config.mjs           # Expressive Code (syntax highlighting) configuration
├── package.json
└── tsconfig.json
```

## Content — writing and editing docs

### File locations

- Docs pages: `src/content/docs/{product}/`
- Partials (reusable snippets): `src/content/partials/{product}/`
- Images: `src/assets/images/{product}/`
- Changelogs: `src/content/changelog/{product}/`

Every folder must have an `index.mdx`. Filenames must be lowercase with dashes between words.

### Allowed file types in `src/content/`

Only `.mdx`, `.json`, `.yml`, `.yaml`, `.txt` files are allowed. The CI will reject anything else. Images must go in `src/assets/images/`, not in `src/content/`.

### Frontmatter

All docs pages require frontmatter. Key fields:

```yaml
---
title: Page Title # Required
description: SEO meta description # Required when pcx_content_type is set
pcx_content_type: how-to # Page type (see below)
sidebar:
  order: 1 # Sort order in sidebar
  label: Custom Label # Override sidebar text
products: # References to src/content/products/ entries
  - workers
difficulty: Beginner # For tutorials: Beginner | Intermediate | Advanced
reviewed: 2025-01-15 # YYYY-MM-DD of last content review
---
```

Valid `pcx_content_type` values: `changelog`, `concept`, `configuration`, `design-guide`, `example`, `faq`, `get-started`, `how-to`, `integration-guide`, `implementation-guide`, `learning-unit`, `navigation`, `overview`, `reference`, `reference-architecture`, `reference-architecture-diagram`, `release-notes`, `solution-guide`, `troubleshooting`, `tutorial`, `video`.

### Writing and style rules

For MDX syntax, links, code blocks, formatting, and writing style, see `.agents/references/style-guide.md`. That file is the canonical agent reference — distilled from the full style guide at `src/content/docs/style-guide/`.

## Components — major APIs

Components are imported from `~/components` in MDX files. Imports must appear after the frontmatter block — forgetting the import is a common mistake.

For full component documentation including props, examples, and mandatory usage rules, see `.agents/references/components.md`.

## Validation — what to run after making changes

> **CI note:** `pnpm run build` will time out in CI environments (GitHub Actions, etc. where `CI=true`). When running in CI, use `pnpm run check` and linters only — do **not** run a full build. The full build is only practical in local development environments.

### Minimum validation for content changes (MDX edits)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/cloudflare-docs](https://github.com/cloudflare/cloudflare-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
