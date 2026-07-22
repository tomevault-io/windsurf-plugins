---
trigger: always_on
description: A ready-to-use, SEO and accessibility-focused Astro starter template with blog and portfolio functionality.
---

# Accessible Astro Starter

A ready-to-use, SEO and accessibility-focused Astro starter template with blog and portfolio functionality.

## Project Overview

- **Type**: Starter theme (static site / blog / portfolio)
- **Output Mode**: Static (SSG)
- **Homepage**: https://accessible-astro-starter.incluud.dev/
- **Repository**: https://github.com/incluud/accessible-astro-starter

Check `package.json` for current version and all dependencies.

### Key Dependencies

- `astro` - Framework (Astro 5.x+)
- `accessible-astro-components` - Core component library
- `tailwindcss` - Styling (v4.x with Vite plugin)
- `@astrojs/mdx` - MDX support
- `@astrojs/sitemap` - XML sitemap generation
- `astro-icon` - Icon system (using Lucide icon set)
- ESLint with `eslint-plugin-jsx-a11y` - Accessibility linting
- Prettier with Astro and Tailwind plugins - Code formatting

## Project Purpose

This is the **flagship starter theme** of the Accessible Astro ecosystem:

- **Target Audience**: Developers building blogs, portfolios, or content-focused websites
- **Features**: Blog with pagination, project portfolio, contact forms, MDX support
- **Showcase**: Demonstrates all components from `accessible-astro-components`
- **WCAG 2.2 AA**: Fully compliant with comprehensive accessibility features
- **Reference Implementation**: Shows best practices for using the component library

## Dev Environment Setup

1. **Install dependencies**:

   ```bash
   npm install
   ```

2. **Start dev server**:

   ```bash
   npm run dev
   # or
   npm start
   ```

   Server starts at `http://localhost:4321`

3. **Build production site**:

   ```bash
   npm run build
   ```

   Output: `./dist/`

4. **Preview production build**:
   ```bash
   npm run preview
   ```

### Workspace Development (Symlinked Components)

This project can work with locally linked `accessible-astro-components`:

- The `astro.config.mjs` automatically detects symlinked packages
- When symlinks are detected, it enables:
  - Auto-reload on component changes
  - Filesystem access to parent directories
  - Symlink preservation in Vite

**To link local components**:

```bash
cd ../accessible-astro-components
npm link
cd ../accessible-astro-starter
npm link accessible-astro-components
```

## Code Style Guidelines

### TypeScript

- Strict mode enabled
- Path aliases configured for cleaner imports:
  - `@components` → `./src/components`
  - `@layouts` → `./src/layouts`
  - `@assets` → `./src/assets`
  - `@content` → `./src/content`
  - `@pages` → `./src/pages`
  - `@public` → `./public`
  - `@post-images` → `./public/posts`
  - `@project-images` → `./public/projects`

### Formatting

- **Prettier** is configured with:
  - `prettier-plugin-astro`
  - `prettier-plugin-css-order`
  - `prettier-plugin-tailwindcss`
- Run format manually: `npx prettier --write .`

### Linting

- **ESLint** configured with:
  - `@typescript-eslint/eslint-plugin`
  - `eslint-plugin-astro`
  - `eslint-plugin-jsx-a11y` (strict accessibility rules)
- Check manually: `npx eslint .`

### Styling

- **Tailwind CSS v4** with Vite plugin
- **SCSS** with custom utilities in `src/assets/scss/`
- **Modern OKLCH** color system with automatic palette generation
- Use logical properties (e.g., `inline-start` instead of `left`)
- Custom properties for theming
- **Atkinson Hyperlegible** font for improved readability

## Accessibility Requirements

This project follows **WCAG 2.2 AA** standards and serves as a reference implementation.

### Essential Practices

1. **Semantic HTML**: Use appropriate elements (`<button>`, `<nav>`, `<main>`, etc.)
2. **Keyboard Navigation**: All interactive elements must be keyboard accessible
3. **Focus Indicators**: Never remove focus outlines without accessible alternatives
4. **Alt Text**: Provide meaningful alt text for images (or use `alt=""` for decorative images)
5. **Color Contrast**: Ensure text has 4.5:1 contrast (3:1 for large text)
6. **ARIA**: Only use ARIA when native HTML is insufficient
7. **Heading Hierarchy**: Maintain proper h1-h6 order
8. **Forms**: Use `<label>`, proper input types, and error messages
9. **Motion**: Respect `prefers-reduced-motion` for all animations

### Testing

- Test with keyboard only (no mouse)
- Test with screen readers (VoiceOver, NVDA, JAWS)
- Use ESLint's jsx-a11y rules to catch common issues
- Check color contrast ratios
- Verify `prefers-reduced-motion` is respected
- Test responsive design on mobile

## File Structure

```
src/
├── components/          # Custom project-specific components
│   ├── Header.astro
│   ├── Footer.astro
│   ├── Navigation.astro
│   ├── SiteMeta.astro       # SEO component
│   ├── ColorContrast.astro  # WCAG contrast checker
│   ├── Featured*.astro      # Content showcase components
│   └── ... (explore for more)
├── content/             # MDX content collections
│   └── projects/        # Project portfolio items
├── layouts/
│   ├── DefaultLayout.astro    # Main layout
│   └── MarkdownLayout.astro   # Layout for MDX content
├── pages/               # Route pages (file-based routing)
│   ├── index.astro      # Homepage
│   ├── accessible-components.astro  # Component showcase
│   ├── blog/
│   │   ├── [...page].astro    # Blog pagination (dynamic route)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [incluud/accessible-astro-starter](https://github.com/incluud/accessible-astro-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
