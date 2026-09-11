---
trigger: always_on
description: This file defines the rules and expectations for AI agents contributing to this Next.js frontend project. All generated
---

# AGENTS.md — Frontend Development Guidelines

This file defines the rules and expectations for AI agents contributing to this Next.js frontend project. All generated
or modified code must comply with these guidelines without exception.

---

## Scope

These rules apply **exclusively to the frontend** — the Next.js application. There is no Next.js backend in this
project. Do not generate API routes, server actions that act as a backend layer, or any server-side business logic
beyond what is strictly needed for rendering.

---

## Project Stack

- **Framework:** Next.js (App Router)
- **Language:** TypeScript
- **Styling:** Follow whatever CSS solution is already in use in the project (Tailwind, CSS Modules, styled-components,
  etc.)

---

## 1. Follow Existing Theme & Styling

- **Never introduce new design tokens.** Colors, spacing, font sizes, border radii, shadows, and breakpoints must come
  from the existing theme — whether that is a Tailwind config, a CSS variables file, a design system, or a theme
  provider.
- **Never hardcode visual values** (`color: #3b82f6`, `padding: 12px`, etc.). Always reference existing tokens or
  utility classes.
- **Match the component style already in the codebase.** If components use Tailwind utility classes, use Tailwind. If
  they use CSS Modules, use CSS Modules. Do not mix styling approaches.
- **Reuse existing UI components** before creating new ones. Check `components/` (or the project's equivalent) for
  buttons, inputs, modals, cards, etc., before building from scratch.
- **Respect dark mode / theming** if it is already set up. New components must support the same theme variants as
  existing ones.

---

## 2. Next.js Best Practices (Vercel / App Router)

### Rendering Strategy

- Default to **Server Components**. Only add `"use client"` when the component genuinely requires browser APIs, event
  listeners, or React state/effects.
- Keep `"use client"` boundaries as **small and deep** in the tree as possible — push interactivity to leaf components.
- Use **`error.tsx`** files at appropriate route segments to handle error states declaratively.
  **No route-level `loading.tsx`** — prefer component-level loading states (skeletons, spinners) to avoid full-page layout shifts and flickering.
- Use **`Suspense`** boundaries around async components for granular loading states.

### Data Fetching

- Fetch data in **Server Components** using `async/await` directly — do not use `useEffect` + `fetch` for initial data
  loading.
- Use Next.js **`fetch` with caching options** (`cache: 'force-cache'`, `next: { revalidate: N }`) intentionally and
  explicitly.
- Avoid fetching the same data in multiple places; lift fetches to the highest relevant Server Component and pass data
  down as props.

### Routing & Navigation

- Use the **App Router** (`app/` directory) exclusively. Do not use the Pages Router.
- Use the `<Link>` component from `next/link` for all internal navigation — never raw `<a>` tags for internal routes.
- Use **Route Groups** `(group)/` to organise related routes without affecting the URL structure.
- Use **Dynamic Segments** (`[slug]`, `[id]`) and **`generateStaticParams`** for statically pre-rendered dynamic routes
  where applicable.

### Performance

- Use **`next/image`** for all images. Always provide `width`, `height`, and meaningful `alt` text.
- Use **`next/font`** to load fonts — never link to external font CDNs in markup.
- Lazy-load heavy client components with `dynamic(() => import(...), { ssr: false })` where appropriate.
- Avoid unnecessary re-renders: stabilise callbacks with `useCallback` and memoize expensive derived values with
  `useMemo` when there is a measurable reason to do so — not by default.

### File & Folder Structure

- Co-locate component-specific files (styles, tests, sub-components) alongside the component.
- Shared, reusable components live in `components/`. Page-specific components live inside their route folder.
- Utility functions live in `lib/` or `utils/`. Constants live in `constants/` or a dedicated file.
- Keep `app/` clean — only route segments, layouts, pages, loading, and error files belong there.

---

## 3. Clean Code Principles

### Function Size & Responsibility

- **Functions must do one thing.** If a function is doing multiple distinct things, split it.
- **Keep functions short** — aim for under 30 lines. If a function exceeds ~40 lines, it is a strong signal it needs to
  be broken down.
- **No deeply nested logic.** Use early returns (guard clauses) to flatten conditionals instead of nesting `if/else`
  blocks.

```ts
// ❌ Avoid
function processUser(user: User) {
    if (user) {
        if (user.isActive) {
            if (user.role === 'admin') {
                // ... 30 more lines
            }
        }
    }
}

// ✅ Prefer
function processUser(user: User) {
    if (!user || !user.isActive){
        return;
    }
    if (user.role !== 'admin') {
       return;
    }
    processAdminUser(user);
}
```

### Naming

- **Names must be intention-revealing.** A reader should understand what a variable, function, or component does without
  needing a comment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igorsyrbu/urlshortener-ui](https://github.com/igorsyrbu/urlshortener-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
