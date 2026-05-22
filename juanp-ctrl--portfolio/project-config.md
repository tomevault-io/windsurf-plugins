---
trigger: always_on
description: You are an expert in React, Next.js App Router, TypeScript, CSS Modules, Tailwind CSS, Framer Motion, and GSAP.
---


# Juan Pablo Portfolio Website - Comprehensive Development Guide

## Role & Approach
You are an expert in React, Next.js App Router, TypeScript, CSS Modules, Tailwind CSS, Framer Motion, and GSAP.

**Workflow Protocol:**
1. **Plan First**: Analyze the task and create a detailed implementation plan
2. **Review**: Critically evaluate the plan for potential issues
3. **Iterate**: Refine the plan if needed before implementation
4. **Execute**: Apply changes only after plan approval
5. **Verify**: Check for linting errors and test the implementation

**Communication:**
- Always ask the user to test the dev environment after significant changes
- Never implement untested or speculative solutions
- Use pnpm exclusively for package management

---

## Project Architecture

### Tech Stack
- **Framework**: Next.js 16.0.1+ (App Router)
- **Runtime**: React 19.2.0+
- **Language**: TypeScript 5.8.3 (strict mode enabled)
- **Package Manager**: pnpm (required)
- **Build Tool**: Turbopack (dev mode)
- **Rendering Strategy**: Client-side rendering with 'use client' directive

### Key Dependencies
- **Animation**: Framer Motion 12.0.0-alpha.1, GSAP 3.13.0
- **Styling**: Tailwind CSS 3.4.17 + CSS Modules
- **UI Components**: Radix UI primitives (shadcn/ui pattern)
- **Internationalization**: next-intl 4.0.0 (cookie-based, en/es)
- **Icons**: Lucide React
- **Utilities**: clsx, tailwind-merge, class-variance-authority

### Project Structure
```
src/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # Root layout (NextIntlClientProvider)
│   ├── page.tsx            # Home page
│   ├── about/
│   │   ├── page.tsx
│   │   └── styles.module.css
│   ├── projects/
│   │   ├── page.tsx
│   │   └── styles.module.css
│   ├── contact/
│   │   └── page.tsx
│   └── not-found.tsx
├── components/             # Feature components with CSS Modules
│   ├── [Component]/
│   │   ├── index.tsx
│   │   └── styles.module.css
│   ├── ui/                 # Reusable UI primitives (shadcn/ui)
│   └── LanguageSwitcher/   # Cookie-based language switcher
├── context/                # React Context providers
│   └── TransitionContext.tsx  # GSAP page transitions
├── hooks/                  # Custom React hooks
├── constants/              # Constants and routes
├── i18n/                   # i18n configuration
│   └── request.ts          # next-intl config
├── lib/                    # Utilities (utils.ts)
└── styles/                 # Global styles
messages/                   # i18n translations (next-intl)
├── en.json                 # English translations (all namespaces)
└── es.json                 # Spanish translations (all namespaces)
public/
├── images/                 # Optimized images (WebP preferred)
├── fonts/                  # Custom fonts
└── cv/                     # Downloadable files
```

---

## Code Quality Standards

### TypeScript Guidelines
- **Strict Mode**: Always enabled
- **Type Safety**: No `any` without explicit comment justification
- **Imports**: Use `@/*` path aliases (configured in tsconfig)
- **Interface vs Type**: Prefer interfaces for objects, types for unions/intersections
- **Null Safety**: Handle null/undefined explicitly

### Component Patterns

#### Client Components (Most Common)
```typescript
'use client'  // Required for interactive components
import styles from './styles.module.css'
import { useTranslations } from 'next-intl'
import { motion } from 'framer-motion'

interface ComponentProps {
  // Props definition
}

export default function Component({ prop }: ComponentProps) {
  const t = useTranslations('namespace')  // next-intl hook
  
  return (
    <div className={styles.container}>
      <h1>{t('title')}</h1>
      {/* Component JSX */}
    </div>
  )
}
```

#### Page Components
```typescript
'use client'  // All pages use client-side rendering
import Header from '@/components/Header'
import PageTransition from '@/components/PageTransition'
import { useTranslations } from 'next-intl'

export default function AboutPage() {
  const t = useTranslations('about')
  
  return (
    <PageTransition>
      <main className="relative w-full overflow-hidden bg-white">
        <Header />
        <h1>{t('title')}</h1>
        {/* Page content */}
      </main>
    </PageTransition>
  )
}
```

### File Naming Conventions
- **Pages**: `page.tsx` in app router directories
- **Components**: PascalCase folders with `index.tsx` + `styles.module.css`
- **Hooks**: camelCase files prefixed with `use` (e.g., `useMedia.tsx`)
- **Types**: PascalCase interfaces/types
- **Constants**: UPPER_SNAKE_CASE for constants, camelCase for files

---

## Styling Standards

### CSS Architecture
**Hybrid Approach**: Tailwind CSS + CSS Modules
- **Tailwind**: Utility classes for layout, spacing, responsive design
- **CSS Modules**: Component-specific styles, animations, complex selectors
- **No Inline Styles**: Avoid unless necessary for dynamic values

### Color System
```css
/* CSS Variables (globals.css) */
:root {
  --black_alternative: #1e1e1e;
  --white_alternative: #fafafa;
  --yellow-alternative: #f2df6b;
  --black: #292826;
  --white: #f0f0f0;
  --yellow: #f8d78f;
}

/* Tailwind Classes */
.black-primary    /* #292826 */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juanp-ctrl/Portfolio](https://github.com/juanp-ctrl/Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
