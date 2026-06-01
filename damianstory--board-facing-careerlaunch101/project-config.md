---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
High-converting sponsorship landing page for Ontario's largest virtual career fair (December 2, 2025). Single-page Next.js application targeting corporate sponsors to book consultations.

## Common Development Commands
```bash
# Development
npm run dev          # Start dev server with Turbo on port 3000
npm run dev:host     # Dev server accessible from network
npm run dev:port     # Dev server on alternate port 3001

# Production
npm run build        # Create production build
npm run start        # Run production server

# Utilities
npm run lint         # Run Next.js linter
npm run clean        # Clear cache and .next folder
npm run restart      # Kill and restart dev server
npm run check        # Check if server is running
```

## Architecture Overview

### Tech Stack
- **Next.js 15.4.5** with App Router and TypeScript
- **Tailwind CSS** with comprehensive design system
- **React 19.1.1** with hooks for state management
- **clsx + tailwind-merge** for dynamic styling
- **Lucide React** for consistent iconography

### Key Design Patterns
1. **Component Composition**: All components use TypeScript interfaces with the `cn()` utility for class merging
2. **Mobile-First**: Every component starts at 320px width with progressive enhancement
3. **Performance First**: Intersection Observer for lazy loading, CSS animations over JavaScript
4. **Accessibility Built-in**: All interactive elements have keyboard support and ARIA labels

### Component Architecture
```typescript
// All components follow this pattern:
interface ComponentProps extends React.HTMLAttributes<HTMLElement> {
  variant?: 'primary' | 'secondary'
  // Additional props
}

export const Component: React.FC<ComponentProps> = ({ variant, className, ...props }) => {
  return (
    <element className={cn(baseStyles, variants[variant], className)} {...props}>
      {/* Content */}
    </element>
  )
}
```

### Section Component Pattern
All section components (Hero, About, Tiers, FAQ) follow this structure:
- Located in `src/components/sections/`
- Use `'use client'` directive for interactivity
- Implement scroll-triggered animations with Intersection Observer
- Include TypeScript interfaces extending `React.HTMLAttributes<HTMLElement>`
- Support className override via `cn()` utility

### State Management Strategy
- **Local State**: React hooks for component state
- **Navigation State**: Intersection Observer tracks active sections
- **Animation State**: CSS-based with JavaScript triggers
- **No Redux/Context**: Single-page app doesn't require global state

### Design System Integration
The design system is implemented through a dual approach:
1. **Tailwind Config**: Extended with project colors, typography, spacing, animations
2. **CSS Custom Properties**: Defined in globals.css for runtime theming and consistency
3. **Utility Classes**: Custom utilities for skeleton loading, focus states, accessibility helpers

### Critical Performance Optimizations
1. **Logo Carousel**: CSS animation with 35s duration, pauses on hover
2. **Statistics Counters**: Only animate when visible via Intersection Observer
3. **Navigation**: Uses `position: sticky` with backdrop blur for performance
4. **Images**: All images must use Next.js Image component with lazy loading
5. **Intersection Observer Pattern**: Used consistently across all sections for animation triggers

## Key Implementation Details

### Navigation Behavior
- Sticky positioning with 80px height (64px mobile)
- Intersection Observer with `-20% 0px -70% 0px` rootMargin for section tracking
- Smooth scroll with offset calculation for sticky header
- Mobile menu prevents body scroll when open
- Dual implementation (desktop/mobile) for optimal UX

### Component Data Patterns
```typescript
// Tier data structure in src/data/tiers.ts
interface TierConfig {
  id: string
  name: string
  price: number
  availability: number | 'unlimited'
  limited?: boolean
  featured?: boolean
  benefits: string[]
}

// FAQ data structure in src/data/faq.ts
interface FAQItem {
  id: string
  question: string
  answer: string
  category?: 'event-details' | 'sponsorship-process'
  keywords?: string[]
}
```

### Animation Architecture
- **CSS Keyframes**: Defined in Tailwind config for optimal performance
- **Intersection Observer**: Triggers animations only when elements are visible
- **Staggered Delays**: Components use incremental delays for smooth sequences
- **Reduced Motion**: Comprehensive support via CSS media queries
- **Timing Standards**: Fast (150ms), Standard (250ms), Slow (400ms)

### Bento Grid System
Recent addition for flexible card layouts:
```typescript
// BentoGrid component provides responsive grid container
// BentoCard component with hover effects and optional backgrounds
// Usage pattern: col-span-3 desktop:col-span-1/2 for responsive layouts
```

## Core Requirements

### Performance Targets
- LCP < 2.5 seconds on mobile networks
- Lighthouse score 90+ across all categories
- Touch targets minimum 48px
- All content lazy-loaded below fold

### Accessibility Standards
- WCAG 2.1 AA compliance mandatory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damianstory/board_facing_careerlaunch101](https://github.com/damianstory/board_facing_careerlaunch101) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
