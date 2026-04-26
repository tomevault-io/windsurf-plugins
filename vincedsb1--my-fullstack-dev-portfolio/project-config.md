---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Next.js 16 personal portfolio website** built with **TypeScript** and **Tailwind CSS**. It's a modern, responsive single-page application showcasing professional work, skills, and experience with dark mode support and scroll-triggered animations.

## Technology Stack

- **Framework:** Next.js 16.0.1 (App Router)
- **Language:** TypeScript 5
- **Styling:** Tailwind CSS 3.3.0 + PostCSS
- **UI Libraries:** React 18, Lucide React, FontAwesome
- **Animations:** AOS (Animate on Scroll)
- **Theme Management:** next-themes
- **Linting:** ESLint (Airbnb preset) + Prettier
- **Deployment:** Vercel (configured via .vercel/)

## Common Development Commands

```bash
# Start development server (localhost:3000)
npm run dev

# Build for production
npm run build

# Run production build locally
npm start

# Run ESLint checks
npm run lint

# Auto-fix linting and formatting issues
npm run fix
```

## Directory Structure

```
src/app/
├── components/           # Feature components (Header, Skills, Projects, Timeline, Footer)
├── integration/          # Integration page with Allaw legal directory embed
├── layout.tsx           # Root layout wrapper
├── page.tsx             # Home/index page
├── Providers.tsx        # Theme provider configuration
├── ThemeSwitcher.tsx    # Dark/light mode toggle component
└── globals.css          # Global styles
```

**Key Pattern:** File-based routing using Next.js App Router. Nested pages are defined by directory structure (e.g., `integration/page.tsx` → `/integration` route).

## Architecture Overview

### 1. **App Router & Page Structure**

- Uses modern Next.js 16 App Router, not Pages Router
- Root layout in `layout.tsx` wraps all pages
- Home page in `page.tsx`
- Nested pages in subdirectories (e.g., `integration/page.tsx`)

### 2. **Component Architecture**

- **Page Components:** Full-page sections (Header, Skills, Projects, Timeline, Footer)
- **Utility Components:** MenuButton (theme toggle), ThemeSwitcher (theme provider)
- Heavy use of `"use client"` directive for client-side interactivity
- Components use React hooks (useState, useEffect, useRef) for state management

### 3. **Theme Management**

```typescript
// Providers.tsx implements theme provider with hydration safety
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
// Returns null until mounted to prevent hydration mismatch
```

Theme switching uses `next-themes` with class-based dark mode. Apply dark styles with `dark:` Tailwind prefix.

### 4. **Styling Approach**

- **Utility-first CSS:** All styling via Tailwind classes
- **Responsive Breakpoints:** Custom in `tailwind.config.ts` (xs: 320px, 2xs: 390px, 3xs: 480px)
- **Dark Mode:** Class-based, configured in Tailwind config
- **Global Styles:** `globals.css` for base styles

### 5. **Animation Pattern**

- AOS library for scroll-triggered animations
- Initialized in component useEffect: `AOS.init({ duration: 200 })`
- Applied via `data-aos` attributes on elements

### 6. **Event Handling Conventions**

- Click-outside detection using refs and event listeners
- Always clean up event listeners in useEffect return
- Use `useRef` for DOM element references

## Configuration Files

- **tsconfig.json:** Strict mode enabled, path alias `@/*` → `./src/*`
- **tailwind.config.ts:** Custom breakpoints, dark mode class-based
- **next.config.js:** Minimal configuration
- **.eslintrc.json:** Airbnb preset, disallows console except warn/error/info
- **.prettierrc:** Default Prettier configuration

## Code Style & Conventions

- **TypeScript Strict Mode:** Required for all new files
- **File Naming:** PascalCase for components (`Header.tsx`), kebab-case for directories (`integration/`)
- **Variables & Functions:** camelCase
- **HTML/CSS IDs & Classes:** Descriptive, semantic
- **Component Props:** Define interfaces for all props
- **Type Safety:** Avoid `any` type; use proper TypeScript interfaces

## Important Notes

### ESLint Configuration

- Extends Airbnb preset with Next.js rules
- Disallows `console` statements except `warn`, `error`, `info`
- Auto-fix with `npm run fix`

### Client-Side Rendering

- This is a primarily client-side portfolio. Most pages use `"use client"` directive
- Minimal server-side logic; no API routes currently configured

### No Testing Framework

- Project currently lacks unit/integration tests
- Consider adding Jest/Vitest if expanding functionality

### Responsive Design

- Mobile-first approach with Tailwind
- Custom breakpoints optimized for various screen sizes
- Test changes on mobile, tablet, and desktop viewports

### External Integrations

- **Integration Page:** Uses Allaw embed script (dynamically loaded with cleanup)
- **Social Links:** GitHub, LinkedIn, Twitter, Malt
- **CV:** PDF file served from public/ directory

## Adding New Features

**New Component:**

1. Create file in `src/app/components/YourComponent.tsx`
2. Add `"use client"` at the top if using hooks
3. Use TypeScript interfaces for props
4. Import and use in appropriate page or parent component

**New Page:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vincedsb1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
