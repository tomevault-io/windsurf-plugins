---
trigger: always_on
description: Multi-zone architecture allows Shipkit applications to be split into multiple Next.js applications while appearing as a single domain to users. This pattern is ideal for:
---

# Multi-Zone Architecture Rules

## Overview

Multi-zone architecture allows Shipkit applications to be split into multiple Next.js applications while appearing as a single domain to users. This pattern is ideal for:

- **Scalability**: Different teams can work on different zones independently
- **Performance**: Each zone can be optimized for its specific use case
- **Deployment**: Zones can be deployed and updated independently
- **Technology Freedom**: Each zone can use different technologies while maintaining consistency

## Zone Configuration Patterns

### Standard Zone Structure
```
domain.com/          → Main app (marketing, dashboard, auth)
domain.com/docs/*    → Documentation zone
domain.com/blog/*    → Blog zone
domain.com/ui/*      → UI component library zone
domain.com/tools/*   → Developer tools zone
```

### Zone Types

#### 1. Main Zone (Primary Application)
- **Purpose**: Core application functionality
- **Contains**: Authentication, dashboard, marketing pages, API routes
- **Routing**: Handles all routes not claimed by other zones
- **Configuration**: Standard Shipkit configuration with multi-zone rewrites

#### 2. Documentation Zone
- **Purpose**: Product documentation, guides, API reference
- **Features**: Search functionality, versioning, navigation tree
- **Content**: MDX files, code examples, tutorials
- **Optimization**: Static generation, fast search indexing

#### 3. Blog Zone
- **Purpose**: Blog posts, announcements, case studies
- **Features**: CMS integration, commenting, social sharing
- **Content**: Articles, author profiles, categories
- **Optimization**: SEO optimization, RSS feeds

#### 4. UI Component Library Zone
- **Purpose**: Component showcase, design system documentation
- **Features**: Interactive component playground, code examples
- **Content**: Component demos, design tokens, usage guidelines
- **Optimization**: Component isolation, visual regression testing

#### 5. Developer Tools Zone
- **Purpose**: Interactive utilities, API explorers, validators
- **Features**: Real-time tools, code generators, testing utilities
- **Content**: Interactive forms, API documentation, utilities
- **Optimization**: Client-side interactivity, tool performance

## Implementation Patterns

### 1. Zone Setup

#### Directory Structure
```
project-root/
├── shipkit/              # Main application
├── shipkit-docs/         # Documentation zone
├── shipkit-blog/         # Blog zone
├── shipkit-ui/           # UI library zone
└── shipkit-tools/        # Tools zone
```

#### Zone Creation Commands
```bash
# Create zones by cloning Shipkit
git clone https://github.com/lacymorrow/shipkit.git shipkit-docs
git clone https://github.com/lacymorrow/shipkit.git shipkit-blog
git clone https://github.com/lacymorrow/shipkit.git shipkit-ui
git clone https://github.com/lacymorrow/shipkit.git shipkit-tools

# Install dependencies for each zone
cd shipkit-docs && bun install --frozen-lockfile
cd shipkit-blog && bun install --frozen-lockfile
cd shipkit-ui && bun install --frozen-lockfile
cd shipkit-tools && bun install --frozen-lockfile
```

### 2. Configuration Patterns

#### Main Zone Configuration (next.config.ts)
```typescript
async rewrites() {
  const multiZoneRewrites = [];

  // Documentation Zone
  if (process.env.DOCS_DOMAIN) {
    multiZoneRewrites.push(
      { source: '/docs', destination: `${process.env.DOCS_DOMAIN}/docs` },
      { source: '/docs/:path*', destination: `${process.env.DOCS_DOMAIN}/docs/:path*` }
    );
  }

  // Add other zones similarly...

  return multiZoneRewrites;
}
```

#### Zone-Specific Configuration
```typescript
// Each zone's next.config.ts
const nextConfig: NextConfig = {
  basePath: '/docs', // or /blog, /ui, /tools
  assetPrefix: '/docs-static', // or /blog-static, etc.

  // Inherit all Shipkit configurations
  ...existingShipkitConfig,
};
```

### 3. Environment Variables

#### Development Environment
```bash
# Main app .env.local
DOCS_DOMAIN=http://localhost:3001
BLOG_DOMAIN=http://localhost:3002
UI_DOMAIN=http://localhost:3003
TOOLS_DOMAIN=http://localhost:3004
```

#### Production Environment
```bash
# Main app production environment
DOCS_DOMAIN=https://docs-shipkit.vercel.app
BLOG_DOMAIN=https://blog-shipkit.vercel.app
UI_DOMAIN=https://ui-shipkit.vercel.app
TOOLS_DOMAIN=https://tools-shipkit.vercel.app
```

## Navigation Patterns

### Inter-Zone Navigation
```tsx
// Use anchor tags for navigation between zones
<a href="/docs/getting-started" className="nav-link">
  Documentation
</a>

// NOT Next.js Link for cross-zone navigation
// ❌ <Link href="/docs/getting-started">Documentation</Link>
```

### Intra-Zone Navigation
```tsx
// Use Next.js Link within the same zone
import Link from 'next/link'

<Link href="/docs/advanced-topics">
  Advanced Topics
</Link>
```

### Shared Navigation Components
```tsx
// Create zone-aware navigation components
const NavLink = ({ href, children, ...props }) => {
  const isExternal = href.startsWith('/docs') ||
                    href.startsWith('/blog') ||
                    href.startsWith('/ui') ||
                    href.startsWith('/tools');

  if (isExternal) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lacymorrow/paperclip-hub](https://github.com/lacymorrow/paperclip-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
