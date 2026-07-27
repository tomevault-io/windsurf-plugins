---
trigger: always_on
description: **Doks** is a modern, minimal Hugo documentation theme built by Thulite. It provides a professional foundation for creating beautiful, high-performance documentation websites with built-in features like full-text search, dark mode, blog support, and internationalization (i18n).
---

# Doks Copilot Instructions

## Project Overview

**Doks** is a modern, minimal Hugo documentation theme built by Thulite. It provides a professional foundation for creating beautiful, high-performance documentation websites with built-in features like full-text search, dark mode, blog support, and internationalization (i18n).

- **Official Website**: https://doks.thulite.io/
- **Documentation**: https://getdoks.org/
- **Repository**: https://github.com/thuliteio/doks

## Tech Stack

### Core Technologies

- **Hugo v0.155.1+** (extended version, required for SCSS support)
  - Static site generator for documentation
  - Uses Goldmark markdown processor with math support (KaTeX/MathJax)
  - Module system (`@thulite/*` packages) for extensibility
- **Node.js v24.13.0+** with npm v11.6.2+
- **Thulite Framework**: Foundational UI framework (Bootstrap 5 compatible)

### Build & Development Tools

- **Vite**: Asset bundling and CSS/JS optimization
- **Prettier**: Code formatting (configured for markdown, JSON, YAML at 2-space tabs; SCSS at 2-space; other files at 4-space)
- **Babel**: JavaScript transpilation
- **PostCSS**: CSS processing with Autoprefixer and PurgeCSS

### Frontend Dependencies

- **@thulite/doks-core**: Core theme layouts and components
- **@thulite/bootstrap**: Bootstrap 5 wrapper
- **@thulite/seo**: SEO optimization module
- **@thulite/images**: Image processing and lazy loading
- **@thulite/inline-svg**: SVG inlining utilities
- **flexsearch**: Client-side full-text search (configured in default params)
- **@tabler/icons**: Icon system
- **basiclightbox**: Image lightbox functionality
- **clipboard**: Clipboard API utilities

### Deployment

- **Netlify**: Configured via `netlify.toml`
  - Build command: `npm run build`
  - Dev command: `npm run dev` (Hugo dev server on port 1313)
  - Functions: Netlify serverless functions support (empty by default)

## Project Structure

```
doks/
├── content/           # Markdown content and pages
│   ├── _index.md     # Homepage
│   ├── privacy.md    # Privacy page
│   ├── blog/         # Blog posts (section: blog)
│   └── docs/         # Documentation pages (section: docs, primary content)
│       ├── guides/   # Guide pages
│       └── reference/ # Reference pages
├── layouts/          # Custom Hugo layouts (override theme defaults)
│   ├── home.html     # Custom homepage layout
│   └── _partials/    # Partial templates
│       ├── head/     # Custom head elements
│       └── footer/   # Custom footer elements
├── assets/           # Static assets processed by Hugo
│   ├── scss/         # SCSS stylesheets
│   │   └── common/   # Custom SCSS (_custom.scss, _variables-custom.scss)
│   ├── js/           # JavaScript files (custom.js for custom logic)
│   ├── images/       # Image assets
│   └── svgs/         # SVG files
├── static/           # Raw static files (copied as-is)
├── config/           # Hugo configuration
│   ├── _default/     # Default configuration
│   │   ├── hugo.toml      # Main Hugo config
│   │   ├── params.toml    # Theme parameters
│   │   ├── module.toml    # Hugo module mounts (integration of @thulite/* packages)
│   │   ├── markup.toml    # Markdown/Goldmark processing settings
│   │   ├── languages.toml # i18n configuration (EN, DE, NL)
│   │   └── menus/         # Navigation menu definitions
│   ├── production/   # Production overrides
│   └── next/         # Future/next version config
├── .changeset/       # Changesets for version management
├── public/           # Built output (generated, do not edit)
├── resources/        # Hugo resource cache (generated)
├── package.json      # Node.js dependencies and build scripts
├── netlify.toml      # Netlify deployment configuration
└── theme.toml        # Theme metadata

**Key Mount Points** (from module.toml):
- `layouts/` custom layouts override `@thulite/doks-core/layouts`
- `home.html` is excluded from doks-core to allow customization
- Data from `@thulite/doks-core/data` merged with local `/data` folder
```

## Build & Development Workflow

### Installation

```bash
npm install
```

This installs all dependencies including Hugo binaries via the node packages.

### Development Server

```bash
npm run dev
```

- Starts Hugo dev server at `http://localhost:1313`
- Hot reload enabled (--disableFastRender --noHTTPCache flags)
- Watches for changes in content, layouts, and assets

### Build for Production

```bash
npm run build
```

- Minifies output (--minify flag)
- Runs garbage collection (--gc flag)
- PurgeCSS removes unused CSS (references hugo_stats.json)
- Outputs to `public/` directory
- Used by Netlify CI/CD

### Code Formatting

```bash
npm run format
```

- Formats all files with Prettier
- Tab widths: 2 spaces (MD, JSON, YAML, SCSS), 4 spaces (default)
- Line ending: CRLF

### Other Commands

```bash
npm run create              # Create new content: npm run create path/to/content.md
npm run preview             # Preview built site
```

## Content Structure & Conventions

### Content Organization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thuliteio/doks](https://github.com/thuliteio/doks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
