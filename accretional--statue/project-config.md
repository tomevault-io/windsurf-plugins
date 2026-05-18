---
trigger: always_on
description: This guide provides a systematic approach to transforming a statue-ssg installation into a fully customized website in a single pass. The project is already initialized with all dependencies, ESM support, and required files.
---

# AGENTS.md - Statue SSG One-Shot Customization Guide

## Purpose
This guide provides a systematic approach to transforming a statue-ssg installation into a fully customized website in a single pass. The project is already initialized with all dependencies, ESM support, and required files.

**This guide supports multiple approaches:**
- **Fully Custom Sites** - Complete control over design and layout
- **Blog/Docs Sites** - Keep default components and markdown content
- **Hybrid Sites** - Mix custom pages with content-driven sections

**Read the requirements carefully** to determine which approach fits the project, then follow the relevant "Option A/B/C" paths throughout this guide.

## Key Principles
1. **Choose the right approach** - Fully custom, content-driven, or hybrid
2. **Design system first** - Define styling patterns before building pages
3. **Component flexibility** - Mix statue-ssg components with custom components
4. **Global components** - Use layout-level components for site-wide elements
5. **Navigation consistency** - Either in layout OR identical on all pages
6. **Single footer location** - Only in layout, never in individual pages
7. **Build configuration** - Add `handleUnseenRoutes: 'ignore'` to avoid prerender errors
8. **Content flexibility** - Keep or delete markdown directories based on needs

## Project Structure (Pre-configured)
- `src/routes/+page.svelte` - Homepage
- `src/routes/+layout.svelte` - Global layout
- `src/routes/about/+page.svelte` - About page (exists)
- `src/lib/components/` - Custom components (create directory)
- `static/assets/` - Images and media (create directory)
- `site.config.json` - Site configuration
- `svelte.config.js` - Build configuration

---

## Phase 1: Static Assets

Place images and media files in `static/assets/`:
```bash
mkdir -p static/assets
# Download or copy images to static/assets/
```

Files in `static/` are served at the root and copied to build output.

---

## Phase 2: Creating Custom Components

### 2.0 Component Architecture
statue-ssg includes default components, but you can create custom components for complete design control.

**Location:** All custom components should be stored in `src/lib/components/`

**Best Practices for Custom Components:**
1. **Use Svelte 4 syntax** - statue-ssg is built on Svelte 4
2. **Make components flexible** - Export props for all configurable options
3. **Use theme variables** - Never hardcode colors; use CSS custom properties
4. **Support slots** - Use default and named slots where appropriate
5. **Include inline SVG** - For icons and simple graphics
6. **Add prop validation** - Set sensible defaults for all props

**Example Component Structure:**
```svelte
<script>
  // Export all configurable props with defaults
  export let backgroundColor = 'var(--theme-bg)';
  export let textColor = 'var(--theme-text)';
  export let padding = '2rem';
  export let showIcon = true;
  // ... more props
</script>

<div style="background-color: {backgroundColor}; color: {textColor}; padding: {padding};">
  {#if showIcon}
    <svg><!-- inline SVG --></svg>
  {/if}
  <slot />
</div>

<style>
  /* Component-scoped styles using theme variables */
</style>
```

**Component Types to Consider:**
- **Gallery** - Image grids with hover effects and captions
- **Footer** - Global footer with contact info and links
- **Navigation** - Custom nav bars with active states
- **Cards** - Content cards with flexible layouts
- **Buttons** - Reusable button components with variants

**Importing Custom Components:**
```svelte
import ComponentName from '$lib/components/ComponentName.svelte';
```

---

## Phase 3: Site Configuration

Update `site.config.json` with your site's information:

**Required sections to customize:**
- `site.*` - Site name, description, URL, author
- `contact.*` - All contact information and addresses
- `social.*` - Social media links (can be empty strings)
- `seo.*` - SEO defaults, title templates, keywords, OG images

**Example:**
```json
{
  "site": {
    "name": "Your Site Name",
    "description": "Your site description",
    "url": "https://yoursite.com",
    "author": "Your Name"
  },
  "seo": {
    "defaultTitle": "Your Site Title",
    "titleTemplate": "%s | Your Site",
    "defaultDescription": "Your site description",
    "keywords": ["keyword1", "keyword2"],
    "ogImage": "/assets/your-image.jpg",
    "twitterCard": "summary_large_image"
  }
}
```

You can access config values in pages with:
```svelte
import siteConfig from '../../site.config.json';
// Use: {siteConfig.site.name}
```

---

## Phase 4: Content Strategy

**Decide your content approach:**

### Option A: Fully Custom Static Site (No Markdown Content)
If building a portfolio, landing page, or fully custom site:
```bash
rm content/example.md
rm -rf content/docs
rm -rf content/blog
rm -rf content/legal
```

### Option B: Blog or Documentation Site (Keep Markdown Content)
If building a blog, docs site, or content-driven site:
- **Keep** `content/blog/` or `content/docs/` directories
- **Delete** placeholder files: `rm content/blog/*.md` or `rm content/docs/*.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [accretional/statue](https://github.com/accretional/statue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
