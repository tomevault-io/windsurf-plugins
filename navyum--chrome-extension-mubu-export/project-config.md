---
trigger: always_on
description: This repository contains a Chrome Extension and a static documentation/blog site for Mubu Exporter.
---

# AGENTS.md

## Project Overview

This repository contains a Chrome Extension and a static documentation/blog site for Mubu Exporter.

- Extension source: `src/`, `manifest.json`, `_locales/`, `icons/`, `asserts/`
- Extension build output: `dist/`
- Static website and blog: `docs/`
- Blog shared assets: `docs/blog/blog-common.css`, `docs/blog/blog-common.js`, `docs/blog/blog-components.js`
- Blog images: `docs/images/`
- SEO/GEO support files: `docs/sitemap.xml`, `docs/llms.txt`, `docs/robots.txt`

The product is a local-first Chrome extension for batch exporting Mubu notes to Markdown, OPML, FreeMind, HTML, Word, PDF, and JSON.

## Common Commands

Install dependencies:

```bash
npm install
```

Build the Chrome extension:

```bash
npm run build
```

Package the extension:

```bash
npm run pack
```

Watch build during extension development:

```bash
npm run build:watch
```

There is no real test suite configured. `npm test` is a placeholder and exits with an error.

For static docs/blog work, opening files under `docs/` is usually enough. If a local server is useful:

```bash
python3 -m http.server 8000 -d docs
```

## Editing Rules

- Prefer editing source files under `src/`, `manifest.json`, `_locales/`, `docs/`, and shared blog components.
- Treat `dist/` as build output. Do not hand-edit `dist/` unless the user explicitly asks.
- Do not overwrite existing image assets unless explicitly asked. Create descriptive sibling filenames instead.
- Preserve existing user changes in the working tree. Do not use destructive git commands such as `git reset --hard` or `git checkout --`.
- Use `rg` / `rg --files` for search.

## Blog And SEO/GEO Rules

Prefer improving or rewriting an existing relevant URL when it already targets the same intent. Create a new URL only when the search intent is clearly different and the new page will not cannibalize an existing keyword.

When creating or rewriting a blog post, update all relevant surfaces:

- The article HTML file in `docs/blog/`
- `docs/blog/index.html` for homepage cards and Blog JSON-LD
- `docs/blog/blog-components.js` for titles and related posts
- `docs/sitemap.xml` with the article URL and current `lastmod`
- `docs/llms.txt` when the article is strategically important for AI citation/GEO
- `docs/blog/cluster-mubu-export/cluster-plan.json` when the keyword cluster status changes

Recommended workflow for new practical tutorials:

- Read nearby articles first and match the existing HTML structure, article voice, metadata style, and component conventions.
- Identify the primary keyword, secondary keywords, target intent, and internal-link targets before writing.
- For platform-specific migration articles, verify current platform behavior from primary/official docs when possible.
- Put the direct answer in the first screen: describe the recommended route in one concise paragraph and one quick-answer block.
- Separate migration from cleanup. Explain the safest order: export first, import second, validate third, then reorganize.
- Add concrete acceptance checks: page count, folder structure, heading/list levels, notes, images, links, permissions, and searchability.
- Include FAQ only when the page answers real search questions, not filler.
- Preserve dates consistently across article metadata, visible article date, homepage cards, JSON-LD, sitemap, and llms updates.

Each important blog post should include:

- Clear SEO title and meta description
- Canonical URL
- Open Graph and Twitter image metadata
- `BlogPosting` JSON-LD
- `BreadcrumbList` JSON-LD
- `HowTo` JSON-LD for practical tutorials when appropriate
- `FAQPage` JSON-LD when the page contains real FAQ content
- A direct quick-answer block near the top
- Practical steps, validation/checklist sections, and internal links
- A body illustration after the quick-answer block when the article has a dedicated blog image
- Related-post links near the end that support the same cluster and user journey

Content requirements for migration tutorials:

- Use a practical, realistic tone. The article should help the reader complete the migration, not just promote the extension.
- State a recommended route and explain when an alternative route is better.
- Mention fallback backups, especially JSON for original Mubu structure and Markdown for portable content.
- Explain what will not migrate automatically: comments, permissions, history, app-specific views, database properties, external image availability, and collaboration state.
- Use tables for format mapping, import-route choices, validation checklists, and permission handoff when they make scanning easier.
- Avoid claiming that target apps preserve every Mubu feature. Prefer "validate", "check", "fallback", "manual handoff", and "import route" language.

Avoid unverifiable or exaggerated claims such as:

- "100%"
- "无损"
- "自动生成双链"
- "自动下载图片"
- "格式保留率"
- Unsupported time/count claims unless they are already sourced in the project

For migration tutorials, be explicit about limitations: permissions, comments, history, external images, and platform-specific import behavior often need manual validation.

## Blog Image Rules

- Preferred blog card/header image size: `1344x768`.
- Preferred format: WebP.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Navyum/chrome-extension-mubu-export](https://github.com/Navyum/chrome-extension-mubu-export) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
