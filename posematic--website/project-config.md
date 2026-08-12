---
trigger: always_on
description: Single-page **landing site** for Posematic: section-based storytelling, scroll-driven reveals, and a waitlist CTA. Built on **Next.js App Router** with a lightweight **MVC split**—pages orchestrate, components render, lib + API hold shared logic and data.
---

# `app/` — Posematic marketing website

Single-page **landing site** for Posematic: section-based storytelling, scroll-driven reveals, and a waitlist CTA. Built on **Next.js App Router** with a lightweight **MVC split**—pages orchestrate, components render, lib + API hold shared logic and data.

## MVC layout

| Layer | Location | Responsibility |
|-------|----------|----------------|
| **Controller** | `app/page.tsx`, future `app/**/page.tsx` | **Composition only**: section order, scroll anchors (`#mission`, `#product`, `#waitlist`), `<Reveal>` wrappers, no inline markup for whole sections. |
| **View** | `app/components/`, `app/globals.css` | **Presentation**: one exported section component per file, Tailwind + design tokens, `next/image`, icons from `lucide-react`. |
| **Model** | `app/lib/`, `app/api/`, colocated constants in components | **Data & rules**: layout tokens, breakpoint helpers, API handlers, section content arrays (e.g. team roster, nav links). |

**Rule:** Keep controllers thin. If a page file grows section markup, move it into a component. If logic is reused or testable in isolation, move it to `app/lib/`.

## Files

| File / folder | What it is (conceptually) |
|---------------|---------------------------|
| **`app/layout.tsx`** | **Root shell**: fonts (`Inter` → CSS var), global metadata, `<html>` / `<body>` classes. No section content. |
| **`app/page.tsx`** | **Home controller**: stacks `<Nav>`, `<main>` sections, `<Footer>`; applies `<Reveal>` to below-the-fold blocks. |
| **`app/globals.css`** | **Design system source of truth**: `@theme inline` breakpoints and color tokens, grain/surface utilities, reveal/hero animation keyframes. |
| **`app/lib/pageLayout.ts`** | **Layout tokens**: `PAGE_EDGE`, `SECTION_EDGE`, `PAGE_MAX`, `SECTION_H2`, `SECTION_LEDE`, `SECTION_PY*`. Import these instead of duplicating spacing/typography strings. |
| **`app/lib/breakpoints.ts`** | **JS breakpoint mirror** of `globals.css`: `screensPx`, `mediaMin()`, `mediaBelow()` for hooks and `matchMedia`. |
| **`app/components/`** | **Section views**: each file = one landing section or shared UI primitive (see table below). |
| **`app/api/waitlist/route.ts`** | **Server handler stub** for email capture; validate input, return JSON. Replace backend when wiring a real provider. |
| **`public/images/`** | **Static assets** referenced as `/images/...` from components. |
| **`next.config.ts`** | **Build config**: image formats, tracing root. Keep minimal. |

### Section components

| Component | Role |
|-----------|------|
| **`Nav.tsx`** | Fixed header, anchor links, mobile menu, compact state after hero scroll. Client. |
| **`Hero.tsx`** | Above-the-fold mission: word cycle, CTA, product image. Client (animation). |
| **`HeroBackdrop.tsx`** | Hero background layer (`next/image`). Server. |
| **`SketchToPose.tsx`** | Product pipeline story (sketch → pose → final). Server. |
| **`Explainer.tsx`** | “What is a posing app?” copy + demo GIF. Server. |
| **`Problems.tsx`** | Pain points / problem framing. Server. |
| **`Vision.tsx`** | Feature vision (`#vision` anchor). Server. |
| **`Team.tsx`** | Team grid; member data colocated as `team[]`. Server. |
| **`ComingSoon.tsx`** | Roadmap / coming-soon band. Server. |
| **`Waitlist.tsx`** | Early-access CTA (currently external Google Form). Server. |
| **`Footer.tsx`** | Site footer. Server. |
| **`Reveal.tsx`** | IntersectionObserver fade-in wrapper; respects `prefers-reduced-motion`. Client. |
| **`PreviewPlayer.tsx`** | Media preview helper (if used by sections). |
| **`GridDistortion.tsx`** | Three.js hero distortion effect. Client; **disabled on mobile** in `HeroBackdrop` due to iOS issues—do not re-enable without testing. |

## Ground truths

### Next.js App Router

- **Default to Server Components.** Add `"use client"` only when the file needs hooks, browser APIs, or event handlers.
- **No `fetch` in client components** for server data—use Server Components, Route Handlers, or Server Actions.
- **Metadata** lives in `layout.tsx` (or per-route `layout`/`page` exports), not in section components.
- **Images**: always `next/image` with meaningful `alt`, sensible `sizes`, and `priority` only above-the-fold (Hero, Nav logo).
- **Links**: in-app anchors use `href="#section-id"`; external CTAs use `target="_blank"` + `rel="noopener noreferrer"`.
- **Package manager**: `pnpm`. Dev uses Turbopack (`pnpm dev`).

### Styling & layout

- **Colors and fonts** come from CSS variables in `globals.css` (`var(--color-brand-purple)`, etc.). Do not introduce one-off hex values when a token exists.
- **Breakpoints**: prefer semantic tiers `tablet:` / `laptop:` / `desktop:` / `wide:` for layout. Use Tailwind `sm:` / `md:` only for finer steps (e.g. Hero typography, nav at 768px).
- **Section chrome**: combine `SECTION_*` + `PAGE_MAX` from `pageLayout.ts`—do not invent new max-width or padding scales per section.
- **Utility classes** defined in `globals.css` (`.grain`, `.surface-liquid`, `.reveal`, `.hero-reveal`) are part of the design system; reuse before adding new global CSS.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Posematic/Website](https://github.com/Posematic/Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
