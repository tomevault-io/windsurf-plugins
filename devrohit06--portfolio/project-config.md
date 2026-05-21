---
trigger: always_on
description: This is an **Astro-based portfolio site** with a multi-framework component architecture. The site uses **component islands** with React, Svelte, and vanilla Astro components coexisting. Static generation is the default, with dynamic API routes for GitHub integration.
---

# AI Agent Instructions for Portfolio Project

## Architecture Overview

This is an **Astro-based portfolio site** with a multi-framework component architecture. The site uses **component islands** with React, Svelte, and vanilla Astro components coexisting. Static generation is the default, with dynamic API routes for GitHub integration.

**Key architectural decisions:**
- **Static-first**: Uses `output: "static"` with server-rendered API endpoints
- **Framework mixing**: React (`client:load`/`client:only`), Svelte (`client:load`), Astro (server-rendered)
- **Theme system**: Pre-rendered theme script (`/public/theme-init.js`) runs before page load to prevent FOUC
- **Animation-heavy**: GSAP with ScrollTrigger drives all scroll animations; cleanup required on navigation

## Critical File Locations

```
src/
├── components/           # Mixed framework components
│   ├── *.astro          # Server-rendered Astro components
│   ├── react/           # React islands (Embla carousels, heatmaps)
│   └── svelte/          # Svelte islands (theme changer, Discord status)
├── content/blog/        # MDX blog posts (Astro content collections)
├── layouts/             # Page layouts with SEO + Analytics
├── lib/                 # Core utilities and data
│   ├── metaData.ts      # Centralized meta descriptions with responsive line breaks
│   ├── projects.ts      # Project data source (use this, not hardcoded arrays)
│   ├── graphql.ts       # GitHub GraphQL queries
│   └── customTransition.ts  # Astro View Transitions config
├── pages/
│   ├── index.astro      # Homepage with GSAP animations
│   ├── blogs/[slug].astro   # Dynamic blog routes with TOC
│   ├── projects/[slug].astro # Dynamic project pages with GitHub API
│   └── api/             # API routes for GitHub data
├── styles/global.css    # CSS custom properties (--bg-primary, --accent-primary, etc.)
└── utils/animations.js  # GSAP animation library
```

## Developer Workflows

### Build & Dev Commands
```bash
bun dev       # Dev server at localhost:4321
bun build     # Static build to dist/
bun preview   # Preview production build
```

### Adding New Content

**Blog posts** (MDX with content collections):
1. Create `.mdx` file in `src/content/blog/`
2. Add frontmatter: `title`, `description`, `date`, `image`, `tags`
3. Dynamic route at `/blogs/[slug].astro` auto-generates pages via `getStaticPaths`

**Projects** (GitHub-integrated):
1. Add entry to `src/lib/projects.ts` array with `githubRepo`, `npmPackage`, `demoLink`
2. Dynamic route at `/projects/[slug].astro` fetches README from GitHub API
3. Uses `marked` for markdown parsing, `ContentWrapper` for rendering

### Theme System
- Theme preference stored in `localStorage.theme` ("dark" | "light")
- `/public/theme-init.js` runs synchronously before DOM render
- CSS variables defined in `src/styles/global.css` under `:root` and `.dark-theme`
- Theme toggle component: `src/components/svelte/ThemeChanger.svelte`

## Project-Specific Conventions

### Component Hydration Strategy
- **`client:load`**: Interactive components needed immediately (theme changer, Discord status)
- **`client:visible`**: Lazy-loaded on scroll (GitHub activity chart in bento grid)
- **`client:only="react"`**: Framework-specific components (Three.js background)
- **Server-rendered**: All `.astro` components by default (SEO-friendly)

### CSS Variable System
All colors use CSS custom properties for theme switching:
```css
var(--bg-primary)      /* Main background */
var(--bg-secondary)    /* Card backgrounds */
var(--text-primary)    /* Main text */
var(--accent-primary)  /* Brand color (blue in light, red in dark) */
var(--accent-primary-rgb)  /* RGB values for alpha variants */
```

### GSAP Animation Patterns
**Critical**: GSAP instances must be cleaned up on navigation to prevent stale ScrollTriggers.

```javascript
// In page scripts
import { cleanupGSAPAnimations, fadeInUp } from '../utils/animations.js';

// Setup animations
document.addEventListener('astro:page-load', () => {
  fadeInUp('.animate-element', { 
    scrollTrigger: { trigger: '.section' } 
  });
});

// REQUIRED: Cleanup before navigation
document.addEventListener('astro:before-preparation', () => {
  cleanupGSAPAnimations();
});
```

Available animation functions: `fadeInUp`, `fadeInLeft`, `fadeInRight`, `scaleIn`, `createParallax`, `floatingAnimation`, `animateMenuText`

### Dynamic Routing Pattern
Follow this pattern for `getStaticPaths`:

```astro
---
import { projects } from '../../lib/projects';

export async function getStaticPaths() {
  return projects.map(project => ({
    params: { slug: project.slug },
    props: { project }
  }));
}

const { project } = Astro.props;
---
```

### Markdown Rendering
- **Blogs**: Use Astro content collections (`getCollection("blog")`) + `render()` method
- **External markdown** (GitHub READMs): Use `marked` package, render with `ContentWrapper.astro`
- ContentWrapper applies `.blog-content` prose styles for consistent typography

### GitHub API Integration
API routes in `src/pages/api/_services/github/` use:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevRohit06/portfolio](https://github.com/DevRohit06/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
