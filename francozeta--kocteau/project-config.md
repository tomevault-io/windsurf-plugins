---
trigger: always_on
description: These rules apply to all work inside this repository. They are inspired by Vercel's Web Interface Guidelines and by Jakub Krehel's design-engineering approach to craft and iteration, but adapted to Kocteau's product direction: a dark, editorial, human-led music review experience.
---

# Kocteau Agent Rules

These rules apply to all work inside this repository. They are inspired by Vercel's Web Interface Guidelines and by Jakub Krehel's design-engineering approach to craft and iteration, but adapted to Kocteau's product direction: a dark, editorial, human-led music review experience.

Kocteau should feel closer to Letterboxd, Apple Music editorial, and a quiet music journal than to a SaaS dashboard or generic social network.

---

## 1. Product Truths

- Kocteau is a music review and taste discovery product.
- The core experience is not “social posting”; it is reviewing music, expressing taste, and discovering what to hear next through other listeners.
- Reviews are the main event.
- The feed is the primary surface, not a generic dashboard.
- For You is the main signed-in home experience.
- Human taste is the source material. Lightweight algorithms route that taste.
- Editorial curation matters, especially during cold-start.
- Social features exist to support discovery, trust, and recommendation quality. They should not overpower reviews.
- Improve the existing product. Do not rebuild the app from scratch unless explicitly asked.

---

## 2. Product Loop

Protect and improve this loop:

1. User enters with email OTP.
2. User completes profile onboarding.
3. User chooses initial taste signals.
4. User reviews a track.
5. User interacts with reviews through likes, bookmarks, comments, or follows.
6. User returns to a personalized For You feed that improves from behavior.

When making changes, ask:

- Does this make reviewing music easier?
- Does this make discovery feel more human?
- Does this improve the For You loop?
- Does this preserve the editorial/minimal feel?
- Does this add clarity without adding noise?

If the answer is no, avoid the change unless explicitly requested.

### Design Engineering Mindset

- Use AI to accelerate iteration, not to replace product judgment.
- Prototype uncertain UI directions quickly, then keep, refine, or discard them fast.
- The human stays in charge of taste, hierarchy, writing, and final polish.
- Build scaffolding first, then tweak, polish, and animate.
- Prefer smaller sequential tasks over one giant ambiguous prompt.
- Understand what the agent changed before building on top of it.

---

## 3. Stack Defaults

Use the existing stack and patterns:

- Next.js App Router
- React
- TypeScript
- Tailwind CSS
- shadcn/ui
- Supabase Auth, Postgres, Storage, RLS, and RPCs
- TanStack Query
- `cn(...)` for class composition
- `motion/react` only when motion improves clarity
- Server routes and Supabase RPCs for write-sensitive flows

Prefer existing project conventions over new abstractions.

For motion-specific decisions, use [docs/ai/motion-rules.md](./docs/ai/motion-rules.md).
For interface craft, visual polish, color, typography, and gesture decisions, use [docs/ai/interface-craft-rules.md](./docs/ai/interface-craft-rules.md).

---

## 4. Visual Direction

Kocteau should feel:

- dark
- editorial
- minimal
- premium
- musical
- quiet
- human
- slightly cinematic through tone and imagery, not decorative effects

The visual language should be mostly monochrome.

Use color with restraint:

- The star/rating accent may use color.
- Album and track covers may provide natural color.
- Avoid extra accent colors unless already part of the product system.
- Do not introduce purple, blue, green, gradient, neon, or SaaS-like color accents without explicit direction.

Prefer:

- restrained contrast
- thin borders
- soft dark surfaces
- stable geometry
- calm spacing
- subtle dividers
- strong typography only where it supports reviews or music identity

Avoid:

- loud gradients
- glow-heavy styling
- oversized shadows
- glassmorphism for decoration
- dashboard-like cards
- excessive badges
- colorful pills
- productivity-app UI patterns

Small details compound into feel. Favor micro-polish that quietly improves the interface:

- use balanced or pretty text wrapping when long headings or copy would otherwise break awkwardly
- use concentric border radii for nested surfaces when padding and radius are both visible
- align optically, not only geometrically, especially in icon + text controls
- use tabular numbers when values update or compare side by side
- keep dark text rendering crisp and calm

---

## 5. Layout Principles

- The primary column should carry the experience.
- Reviews should dominate the hierarchy.
- Secondary rails should support discovery, not compete with the feed.
- The sidebar should remain stable, quiet, and functional.
- Do not over-design navigation.
- Keep surfaces visually integrated instead of stacking too many separate containers.
- Prefer one strong reading flow over multiple competing focal points.
- Keep dimensions stable to avoid layout shift.
- Use empty space intentionally, not as a placeholder for unfinished modules.

For the feed layout:

- Left sidebar = navigation and primary action.
- Center column = search, feed filters, reviews.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [francozeta/kocteau](https://github.com/francozeta/kocteau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
