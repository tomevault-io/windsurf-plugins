---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a modern portfolio website for Tommy Nurwantoro, an Engineering Manager. It's a single-page application (SPA) built with React 19, TypeScript, Vite, Framer Motion, and Tailwind CSS v4, featuring a dark theme with emerald accents, smooth animations, and interactive components.

**Key Architecture:**
- **Component organization:** Reusable UI components in `src/components/ui/` and page-specific sections in `src/components/sections/`
- **Centralized data:** All content data (projects, experiences, education, skills) is in `src/constants/data.ts`
- **Type safety:** Comprehensive TypeScript interfaces in `src/types/index.ts`
- **Custom hooks:** `useInView` for scroll-triggered animations using Intersection Observer

## Development Commands

```bash
npm run dev      # Start development server (runs on http://localhost:5173)
npm run build    # TypeScript compile + Vite build for production
npm run preview  # Preview production build locally
npm run lint     # Run ESLint
```

**Docker:**
```bash
docker build -t myporto .
docker run -p 3001:3001 myporto  # Container listens on port 3001 internally
```

The production Docker container serves the built static files via Nginx on port 3001 (as configured in the Dockerfile).

## Content Management

When updating content, edit these files:

- **`src/constants/data.ts`** - All content data (projects, experiences, education, skills)
- **`src/constants/theme.ts`** - Theme constants for colors, spacing, typography
- **`tailwind.config.js`** - Custom animations and Tailwind extensions

**Important:** Project images are imported in `src/constants/data.ts` and stored in `src/assets/`. When adding new projects, import the image at the top of the file and include it in the project object.

## Component Architecture

### UI Components (`src/components/ui/`)

These are reusable, design-system components:
- **Modal** - Accessible modal with focus management
- **Button** - Variants: primary, secondary, ghost, icon
- **Card** - Container component with consistent styling
- **ProgressBar** - Animated progress bars for skills
- **Section** - Wrapper for page sections with in-view animation
- **AnimatedBackground** - Mouse-tracking gradient background
- **ParticleBackground** - Floating particle effects

### Page Sections (`src/components/sections/`)

These compose the main page:
- **Header** - Hero section with introduction
- **Skills** - Skill categories with progress bars
- **Projects** - Project grid with modal detail view
- **Experience** - Work history timeline
- **Education** - Educational background
- **Contact** - Contact information and links

### Utility Components

- **Navigation** - Fixed nav bar with smooth scroll
- **SchedulePage** - Calendly integration page
- **PaymentPage** - Payment/support page

## Animation System

The app uses multiple animation layers:

1. **Framer Motion animations:** Primary animation system for sophisticated effects
   - Typewriter effects on headings
   - 3D tilt effects on hover
   - Staggered entrance animations with variants
   - Scroll-synchronized timeline animations
2. **Scroll-triggered animations:** The `useInView` hook (src/hooks/useInView.ts) uses Intersection Observer to add `data-in-view` attributes when elements enter viewport (used in Projects component)
3. **Mouse-tracking:** `AnimatedBackground` follows mouse position (managed in App.tsx state)
4. **Custom animations:** Defined in `tailwind.config.js` (gradient animations, floats, pulses)
5. **CSS animations:** Global styles in `src/index.css` (gradient-text animation)

## Framer Motion Integration

The app uses Framer Motion for sophisticated animations:

1. **Custom hooks:** Located in `src/hooks/framer/`
   - `useTypewriter` - Cinematic text typing effects with cursor
     - Parameters: `text`, `speed` (default 50ms), `delay`, `loop`
     - Returns: `displayedText`, `isTyping`, `isComplete`, `reset`
     - Example: Used in Header for typing "Engineering Manager" title
   - `useTilt` - 3D perspective tilt on mouse movement
     - Parameters: `tiltRange` (default 8deg), `smoothing` (default 0.1)
     - Returns: `rotateX`, `rotateY`, `scale` MotionValues, `ref`, `reset`
     - Use with `useMotionTemplate` for transform string
     - Example: Used on profile picture for interactive 3D effect
   - `useTimeline` - Scroll-synchronized timeline animations
     - Parameters: `containerRef`, `itemCount`, `offset`
     - Returns: `progress` (0 to itemCount), `scrollYProgress`, `containerRef`
     - Maps scroll position to item progress for multi-step animations

2. **Component patterns:**
   - Use `motion.div` to wrap elements for animation
   - `AnimatePresence` for enter/exit transitions
   - `variants` for stagger children animations
   - `whileInView` for scroll-triggered animations
   - `useMotionTemplate` for dynamic transform strings combining MotionValues

3. **Performance:**
   - `viewport={{ once: true }}` prevents replay
   - `layout` prop for smooth layout animations
   - `useMotionValue` for tracked values (mouse, scroll)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tommynurwantoro/myporto](https://github.com/tommynurwantoro/myporto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
