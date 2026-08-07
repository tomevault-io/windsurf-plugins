---
trigger: always_on
description: <!-- AUTO-GENERATED from AGENTS.md — do not edit directly.
---

<!-- AUTO-GENERATED from AGENTS.md — do not edit directly.
     Run `bash scripts/sync-agent-rules.sh` to regenerate. -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Website Reverse-Engineer Template

## What This Is
A reusable template for reverse-engineering any website into a clean, modern Next.js codebase using AI coding agents. The Next.js + shadcn/ui + Tailwind v4 base is pre-scaffolded — just run `/clone-website <url1> [<url2> ...]`.

## Tech Stack
- **Framework:** Next.js 16 (App Router, React 19, TypeScript strict)
- **UI:** shadcn/ui (Radix primitives, Tailwind CSS v4, `cn()` utility)
- **Animation:** GSAP 3 (all plugins free) via `@gsap/react` `useGSAP` — the default for scroll-driven, sequenced, split-text, morph, and physics-based motion
- **Icons:** Lucide React (default — will be replaced/supplemented by extracted SVGs)
- **Styling:** Tailwind CSS v4 with oklch design tokens
- **Deployment:** Vercel

## Commands
- `npm run dev` — Start dev server
- `npm run build` — Production build
- `npm run lint` — ESLint check
- `npm run typecheck` — TypeScript check
- `npm run check` — Run lint + typecheck + build

## Code Style
- TypeScript strict mode, no `any`
- Named exports, PascalCase components, camelCase utils
- Tailwind utility classes, no inline styles
- 2-space indentation
- Responsive: mobile-first

## Design Principles
- **Pixel-perfect emulation** — match the target's spacing, colors, typography exactly
- **No personal aesthetic changes during emulation phase** — match 1:1 first, customize later
- **Real content** — use actual text and assets from the target site, not placeholders
- **Beauty-first** — every pixel matters
- **Motion-perfect emulation** — animations are part of the design. Match the target's easing, duration, stagger, and scroll behavior, not just the end state.

## Animation (GSAP)
Reproduce a target's motion with **GSAP**, not hand-rolled CSS, whenever the behavior is scroll-linked, pinned, scrubbed, sequenced (timelines), split-text, morphing, parallax, or physics/inertia-based. Plain CSS transitions are acceptable only for trivial hover/focus color changes.

Conventions (see `docs/GSAP_GUIDE.md` for full patterns — read it before building any animated section):
- Import GSAP and every plugin from **`@/lib/gsap`** (never bare `"gsap"`) — plugins are pre-registered there and SSR-safe.
- Any component that animates begins with `"use client"`.
- Drive **every** animation through the **`useGSAP()`** hook with `{ scope: ref }` — it auto-reverts on unmount and is Strict-Mode safe. Never manually add/remove tweens in a bare `useEffect`.
- For inertia/smooth scrolling (target uses Lenis, Locomotive, or eased scroll), mount **`<SmoothScroller>`** — the GSAP-native (ScrollSmoother) replacement — instead of adding those libraries. Mount it only when the target actually smooth-scrolls.
- Match exact values extracted from the target: ease name (or `CustomEase` for bespoke curves), duration, delay, stagger, and ScrollTrigger `start`/`end`/`scrub`/`pin`. Do not guess.
- Wrap responsive or reduced-motion variants in `gsap.matchMedia()`.

## Project Structure
```
src/
  app/              # Next.js routes
  components/       # React components
    ui/             # shadcn/ui primitives
    icons.tsx       # Extracted SVG icons as React components
    SmoothScroller.tsx # GSAP ScrollSmoother wrapper (mount only for smooth-scroll sites)
  lib/
    utils.ts        # cn() utility (shadcn)
    gsap.ts         # Central GSAP plugin registration + re-exports (import GSAP from here)
  types/            # TypeScript interfaces
  hooks/            # Custom React hooks
public/
  images/           # Downloaded images from target site
  videos/           # Downloaded videos from target site
  seo/              # Favicons, OG images, webmanifest
docs/
  research/         # Inspection output (design tokens, components, layout)
  design-references/ # Screenshots and visual references
scripts/            # Asset download scripts
```

## MOST IMPORTANT NOTES
- When launching Claude Code agent teams, ALWAYS have each teammate work in their own worktree branch and merge everyone's work at the end, resolving any merge conflicts smartly since you are basically serving the orchestrator role and have full context to our goals, work given, work achieved, and desired outcomes.
- After editing `AGENTS.md`, run `bash scripts/sync-agent-rules.sh` to regenerate platform-specific instruction files.
- After editing `.claude/skills/clone-website/SKILL.md`, run `node scripts/sync-skills.mjs` to regenerate the skill for all platforms.

# Website Inspection Guide

## How to Reverse-Engineer Any Website

This guide outlines what to capture when inspecting a target website via Chrome MCP or browser DevTools.

## Phase 1: Visual Audit

### Screenshots to Capture
- [ ] Every distinct page — desktop, tablet, mobile
- [ ] Dark mode variants (if applicable)
- [ ] Light mode variants (if applicable)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rijal-Madan-777/website-cloner](https://github.com/Rijal-Madan-777/website-cloner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
