---
trigger: always_on
description: hyAway is a React + TypeScript frontend application for browsing hydrus network files. It uses:
---

# hyAway Project Instructions

## Project Overview

hyAway is a React + TypeScript frontend application for browsing hydrus network files. It uses:

- **React 19** with TypeScript
- **TanStack Router** for file-based routing
- **TanStack Query** for data fetching
- **Zustand** for client-side state management
- **Zod** for schema validation (API responses, forms)
- **Tailwind CSS v4** for styling
- **Base UI** primitives via shadcn/ui
- **Vite** as the build tool

## Developer Workflows

```bash
pnpm dev          # Start dev server on port 3000
pnpm build        # Production build (vite build && tsc)
pnpm test         # Run Vitest tests
pnpm check        # Format + lint fix
pnpm typecheck    # TypeScript check only
pnpm storybook    # Component playground on port 6006
```

**Adding UI primitives:** Use `npx shadcn@latest add <component>` - components install to `src/components/ui-primitives/`.

## Documentation

Detailed documentation lives in `.github/docs/`. Path-specific instructions in `.github/instructions/` automatically load relevant docs based on the files you're working with.

| Topic               | Document                              | Loaded When Working On                  |
| ------------------- | ------------------------------------- | --------------------------------------- |
| Settings patterns   | `docs/settings-architecture.md`       | `components/settings/`, settings routes |
| File-based routing  | `docs/routing-conventions.md`         | `routes/` files                         |
| Component placement | `docs/component-organization.md`      | `components/` files                     |
| Thumbnail gallery   | `docs/features/thumbnail-gallery.md`  | `thumbnail-gallery/` components         |
| File viewer         | `docs/features/file-viewer.md`        | `file-detail/` components               |
| Tags system         | `docs/features/tags-system.md`        | `tag/` components                       |
| Hydrus API          | `docs/integrations/hydrus-api.md`     | `integrations/hydrus-api/`              |
| TanStack Query      | `docs/integrations/tanstack-query.md` | Query files                             |
| UI primitives       | `docs/ui/primitives.md`               | `ui-primitives/` components             |
| Responsive design   | `docs/ui/responsive-design.md`        | CSS and layout components               |
| CSS patterns        | `docs/ui/css-patterns.md`             | Animations, CSS variables               |

## Quick Reference

### Project Structure

```
src/
├── components/           # Shared components
│   ├── app-shell/        # Header, sidebar, layout
│   ├── file-detail/      # File viewer components
│   ├── page-shell/       # Page layout primitives
│   ├── settings/         # Settings controls
│   ├── tag/              # Tag components
│   ├── thumbnail-gallery/# Gallery components
│   └── ui-primitives/    # Base UI (shadcn/ui style)
├── hooks/                # Shared hooks
├── integrations/         # External integrations
├── lib/                  # Utilities and stores
└── routes/               # File-based routing
    ├── (settings)/       # Settings routes (pathless group)
    └── _auth/            # Protected routes
        ├── (file)/       # File detail routes
        ├── (galleries)/  # Gallery routes
        └── (remote-pages)/ # Pages routes
```

### Import Conventions

- **`@/`** - Alias for `src/`, use for shared components and utilities
- **Relative paths** - Use for route-specific components in `-components/` folders
- **Never import across route groups** - Each route group should be self-contained
- **No barrel files** - Import directly from source files, not `index.ts` re-exports

```tsx
// Shared components: use @/ alias
import { Button } from "@/components/ui-primitives/button";

// Route-specific components: use relative paths
import { FeatureCard } from "./-components/feature-card";
```

### Component Placement

```
Is it a basic UI element (button, input, card)?     → ui-primitives/
Is it used in the app shell (header, sidebar)?      → app-shell/
Is it a page layout primitive (heading, loading)?   → page-shell/
Is it specific to a feature (gallery, tags)?        → feature folder
Is it only used by one route?                       → routes/{group}/-components/
```

When updating a component with a skeleton, **always update the skeleton to match** (same structure, spacing, heights).

### Naming Conventions

| Type       | Convention           | Example                      |
| ---------- | -------------------- | ---------------------------- |
| Files      | kebab-case           | `thumbnail-gallery-item.tsx` |
| Components | PascalCase           | `ThumbnailGalleryItem`       |
| Constants  | SCREAMING_SNAKE_CASE | `GALLERY_SETTINGS_TITLE`     |

### Routing Patterns

- **Pathless groups `()`** - Organize routes without affecting URLs
- **`-` prefix** - Co-locate route-specific files (ignored by router)
- **Layout routes `_`** - Wrap child routes (e.g., `_auth.tsx`)
- **Settings** - Shared controls + thin wrappers pattern

### State Management

| Store         | Purpose          | Location                                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyaway/hyaway](https://github.com/hyaway/hyaway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
