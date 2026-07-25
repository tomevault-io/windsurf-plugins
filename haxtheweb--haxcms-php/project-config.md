---
trigger: always_on
description: This file provides comprehensive instructions for AI coding agents working within this specific HAXcms site. HAXcms sites are built on the HAX (Headless Authoring eXperience) ecosystem and follow specific patterns for content management, theming, and deployment.
---

# AGENTS.md - HAXcms Site Interface

This file provides comprehensive instructions for AI coding agents working within this specific HAXcms site. HAXcms sites are built on the HAX (Headless Authoring eXperience) ecosystem and follow specific patterns for content management, theming, and deployment.

## Site Structure Overview

This HAXcms site follows the standard structure and conventions:

```
site-root/
├── AGENTS.md              # This file - AI agent instructions
├── site.json              # JSON Outline Schema - site structure & metadata
├── pages/                 # Page content (HTML files)
├── files/                 # Media assets and resources
├── theme/                 # Custom theme files
│   ├── theme.html        # Theme template
│   └── theme.css         # Theme styles
├── assets/               # Icons, banners, and static assets
├── index.html            # Main site entry point
├── manifest.json         # PWA manifest
├── package.json          # Node.js dependencies and scripts
└── custom/               # Custom web components and extensions
```

## Core Site Files

### site.json - Site Manifest
The `site.json` file is the heart of every HAXcms site, using **JSON Outline Schema** format:
- **Structure**: Defines page hierarchy, navigation, and content organization
- **Metadata**: Contains site settings, theme configuration, and SEO information
- **Items Array**: Each page is represented as an item with title, location, metadata, and children
- **Validation**: Must conform to JSON Outline Schema standards

### Pages Directory
- **Location**: All page content stored in `pages/` directory
- **Format**: Semantic HTML files (not Markdown)
- **Naming**: Each page has a corresponding `index.html` in its slug-named directory
- **Content**: Rich content using HAX web components and standard HTML

### Files Directory
- **Assets**: All media files, documents, and resources
- **Organization**: Maintain logical file structure for easy management
- **References**: Linked from pages using relative paths

## Content Management

### Adding New Pages
```bash
# Using HAX CLI (recommended)
hax site --title "Page Title" --content "<p>Initial content</p>" --slug "page-slug"

# Manual creation
# 1. Create directory: pages/page-slug/
# 2. Add index.html with semantic HTML content
# 3. Update site.json to include new page in items array
```

### Page Structure
```json
{
  "id": "unique-page-id",
  "title": "Page Title",
  "location": "pages/page-slug/index.html",
  "metadata": {
    "created": 1234567890,
    "updated": 1234567890,
    "description": "Page description for SEO"
  },
  "indent": 0,
  "parent": null,
  "order": 0
}
```

### Content Guidelines
- **Semantic HTML**: Use proper heading hierarchy (h1, h2, h3, etc.)
- **HAX Components**: Leverage available web components for rich interactions
- **Accessibility**: Ensure WCAG 2.0 AA compliance in all content
- **Performance**: Optimize images and media before adding to `files/`

## Theme Development

### Theme Architecture
- **Base Class**: Custom themes should extend `HAXCMSLitElement`
- **Design System**: Use DDD (Design, Develop, Destroy) tokens for consistency
- **Template**: `theme/theme.html` defines the layout structure
- **Styling**: `theme/theme.css` contains theme-specific styles

### Theme Variables
Available template variables (processed by Twig):
- `{{ title }}` - Site title
- `{{ siteTitle }}` - Same as title
- `{{ description }}` - Site description
- `{{ basePath }}` - Site base URL path
- `{{ hexCode }}` - Primary theme color
- `{{ version }}` - HAXcms version

### Theme Development Workflow
1. **Edit theme files** in `theme/` directory
2. **Use DDD tokens** for consistent design (colors, spacing, typography)
3. **Test locally** with `hax serve` or `npm run serve`
4. **Build theme** with `yarn run build` (critical for HAXCMSLitElement themes)
5. **Validate** across different content types and screen sizes

## Development Environment

### Local Development
```bash
# Start development server
hax serve
# or
npm run serve

# Access at http://localhost (port varies)
```

### Available Scripts (package.json)
- `npm start` or `npm run serve` - Start development server
- `npm run dev` - Development mode with enhanced debugging
- `npm run ghpages:build` - Prepare for GitHub Pages deployment

### Dependencies
- **HAXcms Node.js backend** - Content management and build system
- **Web Components** - Access to 250+ HAX web components
- **DDD Design System** - Consistent design tokens and patterns

## HAX Components Integration

### Component Usage
HAXcms sites have access to the full HAX web component library:

```html
<!-- Video player -->
<video-player source="https://youtube.com/watch?v=example"></video-player>

<!-- Image with lightbox -->
<simple-img src="files/image.jpg" alt="Description"></simple-img>

<!-- Grid layouts -->
<grid-plate layout="1-1">
  <div slot="col-1">Left column content</div>
  <div slot="col-2">Right column content</div>
</grid-plate>

<!-- Interactive elements -->
<multiple-choice-question question="What is HAX?">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haxtheweb/haxcms-php](https://github.com/haxtheweb/haxcms-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
