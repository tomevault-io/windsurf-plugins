---
trigger: always_on
description: TeslaNav is a navigation web app optimized for Tesla's in-car browser. Built with Next.js 16 (App Router), React 19, TypeScript 5, and Tailwind CSS 4.
---

# AGENTS.md - TeslaNav Codebase Guidelines

TeslaNav is a navigation web app optimized for Tesla's in-car browser. Built with Next.js 16 (App Router), React 19, TypeScript 5, and Tailwind CSS 4.

## Build, Lint & Test Commands

```bash
bun install              # Install dependencies (package manager: Bun)
bun run dev              # Start dev server (http://localhost:3000)
bun run build            # Production build
bun run start            # Start production server
bun run lint             # Run ESLint across project
bunx eslint <file>       # Lint a specific file
bunx eslint --fix <file> # Auto-fix lint issues in a file
bunx tsc --noEmit        # Type check without emitting output
bun run scripts/test-waze-bounds.ts  # Manual Waze API bounds test (CLI utility)
```

**No test framework is configured.** There are no unit/integration tests. Validation is done via TypeScript strict mode and manual browser testing. Add `?dev=true` to the URL to enable debug overlays.

## Project Structure

```
app/                    # Next.js App Router - pages and API routes
  api/                  # Server-side API routes (route.ts per endpoint)
  admin/                # Admin dashboard (usage stats)
  record/               # GPX track recording page
  view/                 # GPX track playback page
components/             # React components (ui/ for shadcn/ui primitives)
hooks/                  # Custom React hooks (use* prefix, camelCase.ts files)
lib/                    # Shared server/client utilities
  redis.ts              # Upstash Redis client, cache keys, TTLs, rate limits, API tracking
  utils.ts              # cn() = clsx + tailwind-merge
  gpx.ts                # GPX generation, parsing, interpolation
  posthog-server.ts     # Server-side PostHog singleton
types/                  # TypeScript type definitions (one file per domain)
public/                 # Static assets (icons/, cars/, sw.js service worker)
scripts/                # Developer utility scripts (not part of the app)
```

## Code Style Guidelines

### Import Organization

```typescript
"use client";  // 1. Directive first (if needed)

import { useState, useCallback } from "react";  // 2. React
import mapboxgl from "mapbox-gl";               // 3. External packages
import { cn } from "@/lib/utils";               // 4. Internal path-aliased (@/)
import { LocalComp } from "./LocalComp";        // 5. Relative imports
import type { MyType } from "@/types/foo";      // 6. Type-only imports last
```

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Components | PascalCase | `NavigateSearch`, `SettingsModal` |
| Component files | PascalCase.tsx | `Map.tsx`, `FeedbackModal.tsx` |
| Hooks | camelCase `use` prefix | `useGeolocation`, `useWazeAlerts` |
| Hook files | camelCase.ts | `useGeolocation.ts` |
| Types/Interfaces | PascalCase | `WazeAlert`, `RouteData` |
| Type files | camelCase.ts | `waze.ts`, `route.ts` |
| API routes | route.ts | `app/api/directions/route.ts` |
| Constants | UPPER_SNAKE_CASE | `CACHE_TTL`, `RATE_LIMITS` |
| Functions/variables | camelCase | `fetchDirections`, `handleClick` |
| localStorage keys | `teslanav-` prefix | `teslanav-theme`, `teslanav-follow-mode` |

### TypeScript Guidelines

- **Strict mode is enabled** — all code must pass `bunx tsc --noEmit`
- Use `interface` for object shapes that may be extended; `type` for unions and computed types
- Prefer explicit return types on all exported functions
- Use the `type` keyword for type-only imports: `import type { Foo } from "..."`
- No `any` — use `unknown` with narrowing or proper generic types
- Zod (`zod`) is available for runtime validation of external API responses

### Component Patterns

```typescript
"use client";

import { useState, useCallback, forwardRef } from "react";

interface MyComponentProps {
  required: string;
  optional?: number;
  onAction?: (value: string) => void;
}

// Named exports for components (not default exports, except page.tsx files)
export const MyComponent = forwardRef<HTMLDivElement, MyComponentProps>(
  function MyComponent({ required, optional = 0, onAction }, ref) {
    const handleClick = useCallback(() => {
      onAction?.(required);
    }, [onAction, required]);

    return <div ref={ref} onClick={handleClick}>{/* content */}</div>;
  }
);
```

- Page files (`app/**/page.tsx`) use `export default function`
- All other components use named exports: `export const Foo = ...` or `export function Foo`
- Wrap callbacks in `useCallback` with proper dependency arrays
- Use `useRef` for animation frames, timers, DOM elements, and mutable counters that should not trigger re-renders
- Private helper sub-components (not exported) may live at the bottom of a file (e.g., `Toggle`, `CloseIcon`)
- Prefer inline SVG for one-off icons rather than importing from lucide-react

### State Management

- `useState` with lazy initializer for localStorage-persisted preferences:
  ```typescript
  const [theme, setTheme] = useState(() => localStorage.getItem("teslanav-theme") ?? "dark");
  ```
- `useRef` for values that should not trigger re-renders (animation state, timers, previous values)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [R44VC0RP/teslanav.com](https://github.com/R44VC0RP/teslanav.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
