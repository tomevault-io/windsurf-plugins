---
trigger: always_on
description: - The original repository-specific guidance for `gc-da11yn.github.io`
---

# Digital Accessibility Toolkit + Accessibility Agent Team Instructions

This file merges:
- The original repository-specific guidance for `gc-da11yn.github.io`
- The accessibility specialist team guidance introduced from `a11y-agent-team`

## How to Apply These Instructions

When instructions overlap, use this precedence:
1. **Accessibility and WCAG conformance requirements** (must always be satisfied for UI work)
2. **Repository architecture/workflow requirements** (Eleventy, bilingual patterns, scripts)
3. **General implementation preferences** (style/process details)

If a task is backend/scripts/data-only and does not touch user-facing UI, the accessibility specialist workflow is optional.

---

## Repository Context (Original Project Guidance)

This repository hosts the Government of Canada's Digital Accessibility Toolkit, a bilingual static site built with Eleventy that provides accessibility resources to GC employees and the broader community.

### Architecture Overview

#### Eleventy Configuration Architecture
Modern plugin-based system:

```
eleventy/
├── config/                # Legacy modular config
├── plugins/               # Active plugin architecture
│   ├── base-plugin.js
│   ├── collections-plugin.js
│   ├── filters-plugin.js
│   ├── markdown-plugin.js
│   └── registry.js
└── .eleventy.js           # Main orchestrator
```

#### Bilingual Structure
- Parallel language paths: `src/pages/en/` and `src/pages/fr/`
- Locale-driven templates via `{{ locale }}`
- Cross-language links via `toggle` frontmatter
- Internal links must use `{{ pathPrefix }}`

#### Key Directories
- `src/_data/` for global/computed data
- `src/_includes/` for Nunjucks templates/partials
- `src/pages/en` and `src/pages/fr` for content
- `src/main/en|fr` for landing/category pages
- `src/_docs/` for downloadable documents
- `src/_scss/` for Sass styles
- `src/admin/` for Decap CMS

### Core Development Workflows

#### Creating New Pages
Use `npm run newPage` to generate paired English/French pages with valid frontmatter and toggle links.

#### Documenting Major Changes
For major architecture/template/workflow changes, add docs in `docs/implementation/` including scope, changes, testing notes, and bilingual impacts.

#### Decap CMS Notes
- Admin entry point: `/admin/`
- Config: `src/admin/config.yml`
- Uses GitHub backend + Netlify OAuth
- Editorial workflow creates PRs for content changes

### Important Scripts
- `npm start` - dev server
- `npm run build` - production build
- `npm run newPage` - create bilingual pages
- `npm run link-check` - interactive link validation
- `npm run link-check-quick` - automation-friendly link checks
- `npm run spellcheck` - cspell with source mapping

### Content/Template Patterns

#### Frontmatter Conventions
- `toggle` for cross-language pairing
- `subject` and `tags` for taxonomy
- `internalLinks`, `isDraft`, `needsTranslation`, `archived` for alert/visibility behavior
- `toc`/`tocSimple` for generated navigation

#### URL and i18n Rules
- Always build internal URLs with `{{ pathPrefix }}`
- Prefer language-aware data patterns like `{ en: {...}, fr: {...} }`
- Preserve home-page toggle special handling (`home` tag) when editing language switch logic

### Quality Assurance
- Run targeted checks relevant to your changes (build, link-check, spellcheck)
- Preserve `pages-to-review` and changed-pages workflows
- Do not regress bilingual parity between English and French pages

---

## Accessibility-First Development

This workspace enforces WCAG AA accessibility standards for all web UI code.

### Mandatory Accessibility Check

Before writing or modifying any web UI code — including HTML, JSX, CSS, React components, Tailwind classes, web pages, forms, modals, or any user-facing web content — you MUST:

1. Consider which accessibility specialist agents are needed for the task
2. Apply the relevant specialist knowledge before generating code
3. Verify the output against the appropriate checklists

### Available Specialist Agents

Select these agents from the agents dropdown in Copilot Chat, or type `/agents` to browse:

| Agent | When to Use |
|-------|------------|
| accessibility-lead | Any UI task — coordinates all specialists and runs final review |
| aria-specialist | Interactive components, custom widgets, ARIA usage |
| modal-specialist | Dialogs, drawers, popovers, overlays |
| contrast-master | Colors, themes, CSS styling, visual design |
| keyboard-navigator | Tab order, focus management, keyboard interaction |
| live-region-controller | Dynamic content updates, toasts, loading states |
| forms-specialist | Forms, inputs, validation, error handling, multi-step wizards |
| alt-text-headings | Images, alt text, SVGs, heading structure, page titles, landmarks |
| tables-data-specialist | Data tables, sortable tables, grids, comparison tables, pricing tables |
| link-checker | Ambiguous link text, "click here"/"read more" detection, link purpose |
| accessibility-wizard | Full guided web accessibility audit with step-by-step walkthrough |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gc-da11yn/gc-da11yn.github.io](https://github.com/gc-da11yn/gc-da11yn.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
