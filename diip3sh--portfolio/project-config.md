---
trigger: always_on
description: This guide provides essential information for AI agents working with the portfolio codebase - a Next.js dev portfolio website featuring work showcase, experience, and interactive components.
---

# AI Agent Guidelines for Portfolio Website

This guide provides essential information for AI agents working with the portfolio codebase - a Next.js dev portfolio website featuring work showcase, experience, and interactive components.

## Project Overview

Modern Next.js portfolio website with:

- Interactive work showcase with shuffling animations
- Work experience timeline
- Craft/design section
- Custom UI components with animations
- Responsive design with dark/light mode support

### Tech Stack

- **Framework**: Next.js 14+ (App Router)
- **Styling**: Tailwind CSS
- **UI Components**: Custom components with shadcn/ui patterns
- **Package Manager**: Bun
- **Language**: TypeScript
- **Animations**: CSS animations with performance optimizations
- **Deployment**: Vercel

## Project Structure

### Key Directories

- `app/` - Next.js App Router pages and layouts
  - `page.tsx` - Homepage
  - `work/page.tsx` - Work showcase page
  - `craft/page.tsx` - Craft/design page
  - `layout.tsx` - Root layout
  - `globals.css` - Global styles
- `components/` - Shared UI components
  - `ui/` - Reusable UI components (buttons, text, etc.)
  - `work-card-shuffle.tsx` - Interactive work card component
  - `work-experience.tsx` - Experience timeline component
  - `navbar.tsx` - Navigation component
  - `footer.tsx` - Footer component
- `lib/` - Utility libraries and data
  - `data/` - Static data files (work, about, home content)
  - `utils.ts` - Utility functions
  - `svg/` - Custom SVG icon components
- `public/` - Static assets (images, icons, etc.)

### Important Files

- `app/layout.tsx` - Root layout with metadata and fonts
- `app/page.tsx` - Homepage component
- `lib/data/home.ts` - Homepage content and configuration
- `lib/data/work.ts` - Work showcase data
- `lib/data/about.ts` - About section data
- `components/work-card-shuffle.tsx` - Main interactive component
- `components/work-experience.tsx` - Experience timeline
- `tailwind.config.ts` - Tailwind configuration
- `next.config.ts` - Next.js configuration

## Component Architecture

### Core Components

**Work Card Shuffle** (`components/work-card-shuffle.tsx`):
- Interactive card shuffling animation
- Image gallery with smooth transitions
- Responsive grid layout
- Performance-optimized animations

**Work Experience** (`components/work-experience.tsx`):
- Timeline-based experience display
- Smooth scrolling animations
- Responsive design
- Accessibility-compliant

**UI Components** (`components/ui/`):
- Reusable button, text, and layout components
- Consistent styling patterns
- Accessibility features

### Animation System

The project uses optimized CSS animations following these principles:

- **Fast animations**: 0.2s-0.3s duration maximum
- **Ease-out easing**: For natural motion
- **Hardware acceleration**: Transform and opacity properties
- **Reduced motion**: Respects `prefers-reduced-motion`

## Development Guidelines

### Getting Started

```bash
# Install dependencies
bun install

# Start development server
bun dev  # Runs on default Next.js port (3000)

# Build for production
bun run build

# Preview production build
bun run start
```

### Code Standards

- **TypeScript**: Strict mode with proper typing
- **ESLint**: Biome configuration
- **File naming**: kebab-case for components and files
- **Component naming**: PascalCase for React components

### Coding Guidelines

When writing code for this project, follow these principles:

- Write type-safe TypeScript code with explicit types
- Use descriptive variable and function names
- Prefer functional components with hooks
- Implement accessibility features (ARIA labels, keyboard navigation)
- Use Tailwind CSS classes for styling
- Follow React best practices (keys, memoization when needed)
- Keep components small and focused
- Use early returns for cleaner code

### Component Development

1. Create component in `components/` directory
2. Use TypeScript with proper prop types
3. Implement accessibility features
4. Add responsive design with Tailwind
5. Include proper error handling
6. Test component in different screen sizes

## Working with Content

### Portfolio Data Structure

**Data Files** (`lib/data/`):

- `home.ts` - Homepage hero section, featured work, stats
- `work.ts` - Work showcase items with images, descriptions, links
- `about.ts` - Personal information, skills, contact details
- `craft.ts` - Craft/design projects data

### Content Management

- All content is managed through TypeScript data files
- Images are stored in `public/` directory
- SVG icons are React components in `lib/svg/`
- Content supports internationalization-ready structure

### Adding New Work

1. Add work data to `lib/data/work.ts`
2. Place images in `public/work/` directory
3. Update component to handle new data structure
4. Test responsive layout and animations

## Animation Guidelines

### Keep your animations fast

- Default to use `ease-out` for most animations
- Animations should never be longer than 1s (unless illustrative), most should be 0.2s-0.3s

### Performance Optimization

- Use `transform` and `opacity` for animations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diip3sh/portfolio](https://github.com/diip3sh/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
