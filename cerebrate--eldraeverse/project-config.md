---
trigger: always_on
description: This is a Ghost theme based on the Source theme, maintained for the Associated Worlds blog.
---

# Copilot Instructions for Eldraeverse

This is a Ghost theme based on the Source theme, maintained for the Associated Worlds blog.

## Build, Test, and Lint Commands

- **`npm run dev`** or **`yarn dev`** - Start development mode with live reload
  - Watches `.hbs` files, `assets/css/**`, and `assets/js/**` for changes
  - Automatically rebuilds CSS and JS, triggers browser reload
  
- **`npm test`** - Build and validate the theme
  - Runs `gulp build` (pretest hook) to compile CSS and JS to `assets/built/`
  - Then runs `gscan` to check Ghost theme compatibility

- **`npm run zip`** or **`yarn zip`** - Create distributable `.zip` file
  - Runs build first, then packages theme (excludes node_modules, dist, gulpfile.js)
  - Output: `dist/eldraeverse.zip`

- **`npm run test:ci`** or **`yarn test:ci`** - Strict CI version of theme validation
  - Runs build first, then runs `gscan` with `--fatal` and `--verbose` flags
  - Treats warnings as errors (recommended for pre-deployment)

## Architecture

**Eldraeverse is a Ghost theme with the following structure:**

### Template Files (root `.hbs` files)
- **`home.hbs`** - Homepage (overrides default landing page)
  - Uses unified container layout with full sidebar on right
  - Sections (top to bottom): Latest (full post) → CTA → Featured → Other Recent Posts
  - All content in single `gh-main` column alongside sidebar
- **`index.hbs`** - Main index/archive page (full-width, no sidebar)
- **`post.hbs`** - Single post template
- **`page.hbs`** - Static page template
- **`page-random.hbs`** - Intermediate redirect page for `/random/` route
- **`tag.hbs`** - Tag archive page (full-width, no sidebar)
- **`author.hbs`** - Author profile page (full-width, no sidebar)
- **`custom-tag-cloud.hbs`** - Custom page template that renders normal page content plus a tag cloud
- **`default.hbs`** - Base wrapper for all templates

### Partials (`partials/` directory)
- **`components/`** - Reusable component partials
  - **`latest-full-post.hbs`** - Displays most recent post with full content on homepage
    - Includes post title (H3), feature image, metadata, full content, and "View post & comment" link
    - Used in home.hbs with full gh-content styling and paragraph spacing
  - **`featured.hbs`** - Featured posts section (supports `skipContainer` parameter)
  - **`post-list.hbs`** - Post feed with list/grid layouts (supports `skipContainer` and `skipFirst` parameters)
  - **`cta.hbs`** - Call-to-action/newsletter signup (shown on home between Latest and Featured)
- **`typography/`** - Font and text styling partials
- **`icons/`** - Icon SVG partials
- Post card variants: `post-card.hbs`, `related.hbs`, `related-simple.hbs`, `related-two.hbs`
- **`email-subscription.hbs`** - Newsletter signup form
- **`feature-image.hbs`** - Featured image display logic
- **`lightbox.hbs`** - Image lightbox implementation
- **`search-toggle.hbs`** - Search UI toggle

### Assets (`assets/` directory)
- **`css/screen.css`** - Main stylesheet (PostCSS with nested imports)
- **`js/`** - JavaScript source files
  - `lib/` - Library/dependency files (loaded first)
  - Other `.js` files (concatenated after lib)
- **`built/`** - Compiled/minified output (gitignored during development)
- **`fonts/`** - Web fonts
- **`images/`** - Static images
- **`icons/`** - SVG icons for use in templates

### Build Pipeline
The Gulp build system processes:
1. **CSS** - PostCSS imports (easyimport), autoprefixer, and cssnano minification
2. **JS** - Library files first, then application code, concatenated into `source.js`, uglified
3. **HBS files** - Watched for changes, trigger live reload (no processing)

## Key Conventions

### Homepage Layout Architecture

The homepage uses a **unified container layout** where the page template (home.hbs) owns the layout structure once, and components render content without their own container wrappers.

**Layout structure:**
- `<section class="gh-container has-sidebar gh-outer">` - Main container with 16-column grid
  - `<main class="gh-main">` - Left column (12 cols) containing all content sections
    - `Latest` heading + `latest-full-post` component (full post display)
    - `cta` component (newsletter signup)
    - `featured` component (featured posts section)
    - `Other Recent Posts` heading + `post-list` component (feed with 12 posts)
  - `<aside class="gh-sidebar">` - Right column (4 cols) with subscription/support options

**Key parameters for components:**
- `skipContainer=true` - Component renders content without its own `gh-container` wrapper (used by featured and post-list)
- `skipFirst=true` - Post-list skips the most recent post (displayed separately in Latest section above)
- Post fetch limits adjusted when `skipFirst=true`: add 1 to limit to compensate for skipped post

**Pages without sidebar:**
- `index.hbs`, `tag.hbs`, `author.hbs` - Full-width layout, no sidebar (each wraps content in single `gh-container`)
- `post.hbs` - Single post display (not refactored, different layout)

**CSS grid spacing:**
- Direct children of `gh-main` get margin spacing: 64px between sections
- `.gh-feed` gets gap spacing: `var(--grid-gap)` (42px) between post cards

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cerebrate/Eldraeverse](https://github.com/cerebrate/Eldraeverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
