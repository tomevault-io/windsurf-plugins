---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal software developer portfolio website built with Next.js 15, TypeScript, and Tailwind CSS. Features a modern design with multi-theme system (including synthwave/retrowave), interactive Three.js particle animations, Apple Music integration, and optimized performance for showcasing projects and technical expertise.

## Tech Stack

- **Next.js 15** with App Router and Turbopack
- **TypeScript** for type safety
- **Tailwind CSS** with custom design system
- **Framer Motion** for animations
- **Three.js + @react-three/fiber** for 3D particle animations
- **@react-three/drei** for Three.js utilities
- **React Hook Form + Zod** for form validation
- **Lucide React** for icons
- **next-themes** for multi-theme system (6 themes: light, dark, ocean, forest, synthwave, minimal)
- **Geist & Geist Mono fonts** for modern typography
- **React Markdown + Remark GFM** for content rendering
- **JSON-based content management** for blog posts
- **RSS & Sitemap generation** for SEO optimization
- **Jest + React Testing Library** for comprehensive testing (814 tests, 93%+ coverage)

## Development Commands

```bash
# Development
npm run dev                 # Start development server with Turbopack
npm run build              # Build for production (includes RSS & sitemap generation)
npm run export             # Export for static hosting (includes RSS & sitemap generation)
npm start                  # Start production server

# Code Quality
npm run lint               # Run linting
npx tsc --noEmit          # Type checking

# Testing
npm test                   # Run all tests
npm run test:watch         # Run tests in watch mode
npm run test:coverage      # Run tests with coverage report
npm run test:ci            # Run tests for CI/CD pipeline

# SEO Generation
npm run generate-rss       # Generate RSS feed
npm run generate-sitemap   # Generate sitemap
npm run generate-seo       # Generate both RSS and sitemap

# Version Management
npm run version:patch      # Bug fixes (1.0.0 → 1.0.1)
npm run version:minor      # New features (1.0.0 → 1.1.0)
npm run version:major      # Breaking changes (1.0.0 → 2.0.0)
```

## Version Management

This project uses **Semantic Versioning (SemVer)** with the format: `MAJOR.MINOR.PATCH`

### When to Update Versions

**IMPORTANT**: Always analyze changes and update the version number before committing significant work.

- **PATCH** (1.0.x) - Bug fixes, typos, small tweaks
  - Fixing broken functionality
  - Correcting typos in content or code
  - Minor style adjustments
  - Performance improvements without API changes

- **MINOR** (1.x.0) - New features, backwards compatible
  - Adding new components or pages
  - New blog posts or content
  - Enhanced functionality that doesn't break existing features
  - New configuration options

- **MAJOR** (x.0.0) - Breaking changes
  - Removing or significantly changing public APIs
  - Major architectural changes
  - Dependency updates that change behavior
  - Changes requiring user intervention

### Version Bump Workflow

```bash
# Analyze your changes first, then run the appropriate command:
npm run version:patch      # Creates commit, tag, and pushes automatically
npm run version:minor      # Creates commit, tag, and pushes automatically
npm run version:major      # Creates commit, tag, and pushes automatically
```

These commands automatically:
1. Update package.json version
2. Create a git commit with the version bump
3. Create an annotated git tag
4. Push changes and tags to remote

### Manual Version Management

If you need more control:
```bash
npm version patch -m "Fix: description"     # Update version and commit
git push && git push --tags                 # Push changes and tags
```

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── blog/              # Blog pages and layout
│   │   ├── [slug]/        # Dynamic blog post pages
│   │   ├── layout.tsx     # Blog layout component
│   │   ├── page.tsx       # Blog listing page
│   │   └── blog-client.tsx # Client-side blog components
│   ├── globals.css        # Global styles with Tailwind CSS
│   ├── layout.tsx         # Root layout component
│   └── page.tsx           # Home page
├── components/
│   ├── ui/                # Reusable UI components (Button, ThemeToggle)
│   ├── sections/          # Page sections (Hero, About, Projects, Blog)
│   ├── layout/            # Layout components (Navigation, sub-components)
│   ├── blog/              # Blog-specific components (BlogCard, BlogContent, ReadingProgressBar)
│   └── newsletter/        # Newsletter signup components
├── hooks/                 # Custom React hooks
│   ├── useScrollToSection.ts # Navigation with smooth scrolling
│   ├── useBlogFiltering.ts   # Blog filtering logic
│   └── useReadingProgress.ts # Reading progress tracking
├── lib/
│   ├── types.ts           # TypeScript type definitions and utility types
│   ├── utils.ts           # Utility functions (cn helper, formatters)
│   ├── ui-utils.ts        # UI-specific utility functions
│   ├── constants.ts       # Application constants and configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patlehmann1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
