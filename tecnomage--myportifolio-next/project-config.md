---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository, emphasizing best practices for code quality, performance optimization, and exceptional user experience.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository, emphasizing best practices for code quality, performance optimization, and exceptional user experience.

## Project Overview

This is a high-performance personal portfolio website built with Next.js 15.2.0 and React 19, showcasing full-stack development skills and services. The portfolio features:

- Modern responsive design with CSS modules and mobile-first approach
- Interactive SVG animations and optimized particle systems
- Smooth scrolling navigation with accessibility considerations
- Mobile-friendly responsive layout with touch-optimized interactions
- Contact form with progressive enhancement and social media integration
- Skills categorization with semantic structure (Frontend, Backend, DevOps, CMS)

## Development Commands

- `npm run dev` - Start development server (localhost:3000)
- `npm run build` - Build production version with optimization analysis
- `npm start` - Start production server
- `npm run lint` - Run ESLint for code quality
- `npm run lint:fix` - Auto-fix linting issues
- `npm run type-check` - Run TypeScript type checking
- `npm run analyze` - Bundle analyzer for performance optimization

## Tech Stack

- **Framework**: Next.js 15.2.0 (App Router) with static optimization
- **Frontend**: React 19 with Server Components, TypeScript 5
- **Styling**: CSS Modules with custom animations and CSS-in-JS fallback
- **Icons**: React Icons (FA, SI, TB, HI, MD) with tree-shaking
- **Animations**: Framer Motion with reduced motion support, custom SVG animations
- **CMS**: Sanity (configured but not fully implemented)
- **Performance**: Image optimization, lazy loading, code splitting
- **Accessibility**: ARIA labels, keyboard navigation, screen reader support
- **Linting**: ESLint with Next.js rules, Prettier for formatting

## Architecture & File Structure

```
src/
├── app/
│   ├── components/
│   │   ├── ui/              # Reusable UI components
│   │   ├── sections/        # Page sections (Hero, Services, etc.)
│   │   ├── Footer.tsx       # Social links and contact info
│   │   └── Footer.module.css
│   ├── hooks/               # Custom React hooks
│   ├── utils/               # Utility functions and helpers
│   ├── constants/           # App constants and configuration
│   ├── globals.css          # Global styles and CSS variables
│   ├── page.tsx             # Main portfolio page with all sections
│   ├── page.module.css      # Homepage styles and animations
│   └── layout.tsx           # Root layout with optimized font loading
├── lib/
│   ├── sanity.ts           # Sanity CMS client configuration
│   └── performance.ts      # Performance utilities and monitoring
└── public/
    ├── images/             # Optimized images with multiple formats
    └── icons/              # SVG icons and favicons
```

## 🚀 Performance Best Practices

### Core Web Vitals Optimization
- **LCP (Largest Contentful Paint)**: < 2.5s
  - Optimize hero section images with `next/image`
  - Preload critical resources
  - Use efficient image formats (WebP, AVIF)
  
- **FID (First Input Delay)**: < 100ms
  - Minimize JavaScript execution time
  - Use React 19 concurrent features
  - Implement proper code splitting

- **CLS (Cumulative Layout Shift)**: < 0.1
  - Define image dimensions explicitly
  - Reserve space for dynamic content
  - Use CSS containment where appropriate

### Implementation Guidelines
```typescript
// Example: Optimized image component
import Image from 'next/image'

const OptimizedHero = () => (
  <Image
    src="/hero-image.jpg"
    alt="Portfolio hero"
    width={1200}
    height={600}
    priority // Above-the-fold images
    placeholder="blur"
    blurDataURL="data:image/jpeg;base64,..."
  />
)
```

### Code Splitting Strategy
- Route-based splitting (automatic with App Router)
- Component-based splitting for heavy components
- Dynamic imports for non-critical features
```typescript
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <ComponentSkeleton />,
  ssr: false // If not needed for SEO
})
```

### Bundle Optimization
- Tree-shaking enabled for all dependencies
- Minimize bundle size with webpack-bundle-analyzer
- Use ES modules for better tree-shaking
- Implement service worker for caching strategy

## 💻 Code Quality Best Practices

### TypeScript Standards
```typescript
// Strict type definitions
interface ServiceCard {
  readonly id: string;
  title: string;
  description: string;
  technologies: readonly Technology[];
  performance: {
    priority: 'high' | 'medium' | 'low';
    renderOrder: number;
  };
}

// Use branded types for better type safety
type EmailAddress = string & { readonly brand: unique symbol };
```

### Component Architecture
```typescript
// Compound component pattern example
const Navigation = {
  Root: NavigationRoot,
  Item: NavigationItem,
  Menu: NavigationMenu
} as const;

// Usage with proper composition
<Navigation.Root>
  <Navigation.Menu>
    <Navigation.Item href="#services">Services</Navigation.Item>
  </Navigation.Menu>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tecnomage) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
