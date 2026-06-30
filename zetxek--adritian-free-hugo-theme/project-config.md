---
trigger: always_on
description: This is a **Hugo theme** for personal websites/portfolios, not a standalone site (although a example site is provided, to demonstrate the usage of the theme itself). The theme provides layouts, styling, and functionality that users import via Hugo modules or as a git submodule.
---

# Adritian Hugo Theme - AI Coding Instructions

## Project Overview

This is a **Hugo theme** for personal websites/portfolios, not a standalone site (although a example site is provided, to demonstrate the usage of the theme itself). The theme provides layouts, styling, and functionality that users import via Hugo modules or as a git submodule.

**Technology Stack:**
- **Static Site Generator:** Hugo (v0.153+ extended version required)
- **CSS Framework:** Bootstrap 5 (v5.3.8)
- **CSS Preprocessor:** SCSS/Sass
- **JavaScript Libraries:** Fuse.js (search), DOMPurify (XSS protection)
- **Testing:** Playwright (E2E tests)
- **Package Manager:** npm
- **Templating:** Hugo Templates (Go templates)
- **Build Tools:** Hugo's built-in asset pipeline, PostCSS, Autoprefixer

## Project Architecture

This is a **Hugo theme** for personal websites/portfolios, not a standalone site. The theme provides layouts, styling, and functionality that users import via Hugo modules or as a git submodule.

The theme provides a example site (`exampleSite/`) that demonstrates all features and serves as a starting point for users. It is also used for the integration tests.

**Key Structure:**
- `layouts/` - Hugo templates (partials, shortcodes, content types)
- `assets/scss/` - Bootstrap 5-based styling with theme customizations
- `exampleSite/` - Demo content that gets distributed to users via `adritian-theme-helper` and serves as a github template repository
- `archetypes/` - Content templates for different content types
- `i18n/` - Multilingual translations (en/es/fr)

## Content Architecture & Custom Types

The theme implements **5 custom content types** with specific frontmatter:

1. **Experience** (`content/experience/`) - Job history with `jobTitle`, `company`, `location`, `duration`
2. **Skills** (`content/skills/`) - Technical skills with progress bars and categories
3. **Blog** (`content/blog/`) - Posts with sidebar/default layouts
4. **Client-work** (`content/client-work/`) - Portfolio pieces
5. **Testimonial** (`content/testimonial/`) - User testimonials

**Critical:** Each content type has a corresponding archetype in `archetypes/` and layout in `layouts/`. Always maintain this trio when adding new types.

## Development Workflows

### Local Development
```bash
# Install dependencies
npm install                 # Installs Bootstrap + theme-helper
# Theme development (testing with exampleSite)
npm run serve 
```

### Testing
```bash
npm test                   # Full test suite
npm run test:e2e           # Playwright E2E tests
npm run test:e2e:no-menus  # Tests without navigation (for menu-less configs)
```

**Test Coverage:** E2E tests validate theme switching, language switching, content rendering, search functionality, and HTML validation across all major features.

### Performance Optimization
```bash
npm run build:performance  # Optimized build with critical CSS
node scripts/optimize-performance.js  # Performance analysis
```

## Styling System

**SCSS Architecture:** Modular approach extending Bootstrap 5
- `assets/scss/adritian.scss` - Main entry point importing all partials
- `assets/scss/_variables.scss` - Theme color/sizing variables
- Individual component files: `_navbar.scss`, `_blog.scss`, `_experience.scss`, etc.

**Theme Colors:** Uses CSS custom properties with `$base-color` as primary. Dark/light themes switch via CSS variables, not separate stylesheets.

## Multilingual (i18n) Implementation

**Translation Strategy:**
- Static strings: `i18n/*.yaml` files (navigation, UI elements)
- Content: Hugo's file-based translation (`content/en/`, `content/es/`, etc.)
- **Post-v1.7.0:** Prefers shortcode-based content over i18n strings for flexibility

## Shortcode System

**Primary Content Method:** Pages built with shortcodes rather than traditional Hugo sections.

Key shortcodes in `layouts/shortcodes/`:
- `experience-section` - Displays limited experience list (controlled by `homepageExperienceCount`)
- `experience-list` - Print-friendly experience list
- `contact-section` - Formspree integration
- `showcase-section` - Portfolio display
- `about-section`, `text-section` - Content blocks

**Pattern:** Most homepage sections use shortcodes for maximum customization flexibility.

## Module Distribution Model

**Critical Understanding:** This theme is distributed via:
1. **Hugo Module** (recommended) - `hugo mod get github.com/zetxek/adritian-free-hugo-theme`
2. **Theme Helper Package** - `@zetxek/adritian-theme-helper` downloads demo content
3. **Git Submodule** (legacy) - Direct theme inclusion

**Bootstrap Process:**
```bash
hugo mod get -u github.com/zetxek/adritian-free-hugo-theme
hugo mod npm pack
npm install  # Installs Bootstrap + theme-helper
./node_modules/@zetxek/adritian-theme-helper/dist/scripts/download-content.js
```

## Configuration Patterns

**Hugo Config (`hugo.toml`):**
- Performance optimizations: `[minify]`, `[imaging]`, `[markup]` sections
- Custom output formats for content types (footer content excluded from HTML generation)
- Module imports section for theme loading


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zetxek/adritian-free-hugo-theme](https://github.com/zetxek/adritian-free-hugo-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
