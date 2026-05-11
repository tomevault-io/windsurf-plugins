---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RESQIO Fire Safety Hub is a marketing/landing page website for a comprehensive fire department management system. Built with Vite, React, TypeScript, and shadcn/ui components, this site showcases various modules and features of the RESQIO platform for German-speaking fire departments (Feuerwehr).
## Development Commands

```bash
# Install dependencies (uses npm, but project supports bun as well)
npm i

# Start development server (Vite dev server on port 8080)
npm run dev

# Build for production
npm run build

# Build in development mode
npm run build:dev

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Architecture Overview

### Tech Stack
- **Build Tool**: Vite 5.x with React SWC plugin for fast compilation
- **Framework**: React 18.3 with TypeScript 5.8
- **Routing**: React Router DOM v6 (client-side routing)
- **UI Framework**: shadcn/ui (Radix UI primitives + Tailwind CSS)
- **Styling**: Tailwind CSS with custom HSL-based theming
- **State Management**: TanStack Query for server state
- **Forms**: React Hook Form with Zod validation

### Project Structure

```
src/
├── components/          # Reusable components
│   ├── ui/             # shadcn/ui components (auto-generated, don't manually edit)
│   ├── Header.tsx      # Main navigation with scroll detection
│   ├── Footer.tsx      # Site footer
│   └── *Section.tsx    # Landing page sections (Hero, Features, Pricing, etc.)
├── pages/              # Route pages
│   ├── Index.tsx       # Main landing page (composes all sections)
│   ├── ModulDetail.tsx # Dynamic module detail pages
│   ├── Impressum.tsx   # Legal imprint
│   ├── Datenschutz.tsx # Privacy policy
│   └── NotFound.tsx    # 404 page
├── data/
│   └── module-data.ts  # Central module definitions (all features/modules)
├── hooks/              # Custom React hooks
├── lib/
│   └── utils.ts        # Utility functions (cn for className merging)
└── App.tsx             # Root component with providers and routing
```

### Key Design Patterns

1. **Module Data Architecture**: All RESQIO modules are defined in `src/data/module-data.ts`. This single source of truth contains:
   - Module metadata (title, descriptions, keywords)
   - Benefits and features lists
   - Technical details
   - Icon and color associations
   - SEO metadata

2. **Route Structure**:
   - `/` - Main landing page (Index.tsx)
   - `/modul/:slug` - Dynamic module detail pages using module-data.ts keys
   - `/impressum` - Legal imprint
   - `/datenschutz` - Privacy policy
   - `*` - Catch-all 404 route

3. **Component Composition**: The Index page is composed of discrete section components (HeroSection, FeaturesSection, etc.) arranged sequentially. Each section is self-contained with its own styling and data.

4. **Scroll-based Navigation**: Header.tsx implements smooth scrolling to section IDs on the homepage using the `scrollToSection` function. Navigation links scroll to anchored sections rather than navigate to new routes.

5. **Dynamic Meta Tags**: ModulDetail.tsx updates document title, description, and keywords dynamically based on the module being viewed.

## Styling System

### Tailwind Configuration
- **Base Color**: Slate
- **Theme System**: HSL-based CSS variables (see `src/index.css`)
- **Custom Fonts**:
  - Sans: Poppins
  - Serif: Merriweather
  - Mono: JetBrains Mono
- **Custom Shadows**: Defined via CSS variables (--shadow-xs through --shadow-2xl)

### Color Palette
The site uses a custom color system defined in tailwind.config.ts with semantic naming:
- `background`, `foreground` - Base colors
- `primary`, `secondary` - Brand colors
- `muted`, `accent` - Supporting colors
- `card`, `popover` - Component backgrounds
- Each module in module-data.ts can have its own color (blue, amber, red, slate, etc.)

### Path Alias
- `@/` maps to `./src/` (configured in vite.config.ts and tsconfig.json)

## Important Files

### module-data.ts
Central data source for all RESQIO modules. When adding or modifying module information:
- Use the ModuleData interface
- Include all required fields: title, shortDesc, longDesc, benefits, features, icon
- Optional: technicalDetails, keywords (for SEO), color
- Slug is the object key in the modules record

### Header.tsx
- Implements sticky header with scroll detection
- Background blur and styling changes on scroll
- Scroll progress bar at bottom
- Mobile responsive menu
- Uses `scrollToSection` for smooth scrolling to anchored sections

### App.tsx
- Sets up QueryClient for TanStack Query
- Wraps app in TooltipProvider, Toaster, and Sonner
- Defines all routes with BrowserRouter
- **CRITICAL**: All custom routes must be added ABOVE the catch-all `*` route

## shadcn/ui Components

This project uses shadcn/ui components located in `src/components/ui/`. These are:
- Auto-generated via the shadcn CLI
- Based on Radix UI primitives
- Styled with Tailwind CSS
- Should not be manually edited; use the CLI to update/add components

Configuration: `components.json`

## Content Language

All user-facing content is in German (Deutsch). Maintain German language for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markusstraubmbc/fire-safety-hub](https://github.com/markusstraubmbc/fire-safety-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
