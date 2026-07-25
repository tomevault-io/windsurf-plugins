---
trigger: always_on
description: This file provides guidance to AI coding agents working with the helm.sh website repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with the helm.sh website repository.

## Overview

This is the official Helm project website (helm.sh) - a Docusaurus static site serving Helm documentation, blog, and community resources. The site supports multiple languages and versions, and is automatically deployed to Netlify.

### Technology Stack
- Docusaurus (static site generator)
- React (component framework)
- Node.js/Yarn (package management)
- Netlify (hosting and deployment)

## Quick Start

```bash
# Install dependencies
yarn install

# Start development server
yarn start

# Build for production
yarn build

# Serve production build locally
yarn serve
```

## Repository Structure

### Content Organization
- `docs/` - Current version documentation (unversioned)
- `versioned_docs/version-N/` - Versioned documentation snapshots
- `blog/` - Blog posts and announcements
- `i18n/{lang}/` - Internationalized content for supported languages
- `src/` - React components and custom pages
- `static/` - Static assets (images, files)

### Configuration
- `docusaurus.config.js` - Main Docusaurus configuration
- `sidebars.js` - Documentation sidebar structure
- `versions.json` - Available documentation versions
- `netlify.toml` - Netlify deployment configuration

### Languages
Supported: English (en), German (de), Spanish (es), French (fr), Japanese (ja), Korean (ko), Portuguese (pt), Russian (ru), Ukrainian (uk), Chinese (zh)

## Build and Test Commands

### Development
```bash
# Start dev server (usually port 3000)
yarn start

# Start with specific locale
yarn start --locale ko

# Clear cache if needed
yarn clear
```

### Production Build
```bash
# Build all locales
yarn build

# Build specific locale
yarn build --locale en

# Build without minification (faster for testing)
yarn build --no-minify
```

### Quality Checks
```bash
# Type checking (if TypeScript is added)
yarn typecheck

# Link checking
make check-links-ci

# Spell checking
typos
```

## Architectural Documentation

This codebase includes an `ARCHITECTURAL_DECISIONS.md` document that explains key architectural decisions made during the Docusaurus implementation. When implementing new features or making significant changes:

1. **Document architectural decisions** in `ARCHITECTURAL_DECISIONS.md` with clear reasoning
2. **Include requirements** that drove the decision
3. **Link to relevant Docusaurus documentation** when applicable
4. **Keep explanations concise** but comprehensive enough for future contributors

This helps maintain consistency and guides future development decisions.

### Writing Guidelines for ARCHITECTURAL_DECISIONS.md

When updating architectural decisions:
- **Describe current state** - Document what exists now, not proposals or ideas
- **Keep it concise** - One paragraph per topic, link to external docs instead of repeating them
- **Focus on the "why"** - Explain decisions that aren't obvious from the code
- **Help contributors** - Write for developers who need to understand the codebase quickly
- **Avoid duplication** - Link to Docusaurus docs rather than explaining Docusaurus features

Example: Don't explain what CSS modules are, but DO explain why we chose them over other styling approaches for this specific project.

## Content Management

### Documentation

#### Adding/Editing Docs
1. Edit files in `docs/` for current version
2. For versioned docs, edit in `versioned_docs/version-N/`
3. Frontmatter format:
```yaml
---
title: "Page Title"
sidebar_label: "Short Label"
sidebar_position: 1
---
```

#### Creating New Versions
```bash
# Create new version snapshot
yarn docusaurus docs:version 3.18.0
```
This creates:
- `versioned_docs/version-3.18.0/` - Snapshot of current docs
- `versioned_sidebars/version-3.18.0-sidebars.json` - Sidebar config
- Updates `versions.json`

#### CLI Reference Documentation
Located in `docs/helm/` (and versioned equivalents). To update:
1. Uninstall all helm plugins: `helm plugin uninstall`
2. Navigate to appropriate docs directory
3. Run: `HOME='~' helm docs --type markdown --generate-headers`
4. Commit changes

### Blog Posts

Create in `blog/` directory with naming: `YYYY-MM-DD-slug/index.md`

Frontmatter format:
```yaml
---
title: "Post Title"
authors:
  - name: Author Name
    url: https://author.link
tags: [tag1, tag2]
---

Post summary appears here.

<!--truncate-->

Full post content here.
```

Images go in the same directory as the blog post or in `blog/assets/`.

### Internationalization

#### Adding Translations
1. Extract strings: `yarn write-translations --locale ko`
2. Translate files in `i18n/{locale}/`
3. Content structure:
   - `i18n/{locale}/docusaurus-plugin-content-docs/` - Docs translations
   - `i18n/{locale}/docusaurus-plugin-content-blog/` - Blog translations
   - `i18n/{locale}/code.json` - UI strings

#### Translation Guidelines
- Maintain consistent terminology across versions
- Test with `yarn start --locale {locale}`
- Ensure all navigation and UI elements are translated

## Code Style and Conventions

### Markdown
- Use semantic line breaks (one sentence per line preferred for diffs)
- Code blocks should specify language: ```yaml, ```bash, ```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helm/helm-www](https://github.com/helm/helm-www) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
