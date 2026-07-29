---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Hinode is a Hugo theme for documentation and blog sites built on Bootstrap 5. It uses Hugo's module system to manage dependencies. The theme is designed for performance, security (with CSP headers), and SEO.

**Version 2 (templatev2 branch)** is a minimal core theme. Optional extensions like mod-blocks add features like pre-built Bookshop components for page building.

## Common Development Commands

### Development Server

```bash
npm start                    # Start Hugo server with module vendoring
npm run start:example        # Start server using exampleSite
npm run start:prod          # Start server in production mode
npm run start:example:prod  # Start exampleSite in production mode
```

### Building

```bash
npm run build               # Build site with minification
npm run build:example       # Build exampleSite
npm run build:debug         # Build with debug output
npm run build:headers       # Generate Netlify/server headers
```

### Linting & Testing

```bash
npm test                    # Run all linters
npm run lint                # Run all linters
npm run lint:scripts        # ESLint for JavaScript (assets/js)
npm run lint:styles         # Stylelint for SCSS
npm run lint:markdown       # Markdownlint for Markdown files
```

### Module Management

```bash
npm run mod:vendor          # Vendor Hugo modules to _vendor/ (skips when _vendor is up to date)
npm run mod:vendor:force    # Re-vendor Hugo modules unconditionally
npm run mod:update          # Update all Hugo modules
npm run mod:tidy            # Clean up unused module dependencies
npm run mod:clean           # Remove module cache
```

### Maintenance

```bash
npm run clean:public        # Remove generated public/ directories
npm run clean:install       # Remove node_modules and package-lock.json
npm run upgrade             # Update npm and Hugo module dependencies
```

## Architecture

### Hugo Module System

The theme uses Hugo's module system extensively. All modules are vendored to `_vendor/` for reproducible builds. Key modules include:

**Core modules (always loaded):**

- `mod-bootstrap` - Bootstrap 5 framework
- `mod-utils` - Utility functions and helpers (GetPadding, GetBreakpoint, LogWarn, InitArgs, etc.)
- `mod-flexsearch` - Full-text search functionality
- `mod-fontawesome` - Icon support

**Optional modules:**

- `mod-blocks` - Pre-built Bookshop blocks for page building (NOT loaded by default in v2)
- `mod-katex`, `mod-mermaid`, `mod-leaflet`, `mod-lottie` - Feature modules

Module configuration is in `config/_default/hugo.toml` under `[module.imports]`. Always run `npm run mod:vendor` after module changes.

### Partial Ownership (v2 Architecture)

**Hinode owns (core partials):**

- `assets/card-group.html`, `assets/nav.html`, `assets/video.html`, `assets/table.html`, `assets/timeline.html` - Used by Hinode shortcodes
- `assets/live-image.html`, `assets/live-pages.html` - Used by Hinode templates
- `assets/section-title.html` - Section heading utility (used by Hinode pages and mod-blocks components)
- All `mod-utils` utilities (GetPadding, GetBreakpoint, LogWarn, InitArgs, etc.)

**mod-blocks owns (block-specific partials):**

- `assets/hero.html`, `assets/contact.html`, `assets/faq.html`, `assets/menu.html`, `assets/testimonial-carousel.html`, `assets/preview.html`
- `utilities/section.html` - Component wrapper
- `page/contact.html` - Contact page template

**Dependency flow:**

```text
Hinode v2 (core theme)
  ├── mod-utils (GetPadding, LogWarn, etc.)
  ├── Shared partials (card-group, video, table, section-title, etc.)
  └── Does NOT import mod-blocks by default

mod-blocks v1.1+ (optional extension)
  ├── 16 Bookshop components
  ├── Block-specific partials (7 files)
  └── Depends on Hinode v2 (inherits section-title from Hinode)
```

### Component Library (Bookshop) - Optional via mod-blocks

Bookshop components are provided by the optional **mod-blocks** module. When installed, components live in `component-library/components/`. Each component has:

- `*.hugo.html` - Hugo template
- `*.scss` - Component styles
- `*.bookshop.yml` - Schema definition

Components are mounted to multiple locations via `hugo.toml`:

- `layouts/partials/bookshop/` - Templates
- `data/structures/` - Schemas
- `assets/scss/modules/bookshop/` - Styles

#### Bookshop Component Architecture

Bookshop components follow a two-layer architecture:

**1. Component partial** (e.g., `layouts/partials/assets/preview.html`):

- Contains the core component logic and rendering
- Uses **component-specific arguments** (e.g., `url`, `device`, `heading` for preview)
- These arguments should be defined in the component's structure file

**2. Bookshop wrapper** (e.g., `component-library/components/preview/preview.hugo.html`):

- Calls the component partial with component-specific arguments
- Wraps output with `utilities/section.html` for section-level styling
- Passes **section arguments** to the wrapper (e.g., `id`, `background`, `width`, `justify`, `wrapper`, `fluid`, `theme`, `cover`, `overlay-mode`, `section-class`, `bg-class`)

**Important distinctions:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gethinode/hinode](https://github.com/gethinode/hinode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
