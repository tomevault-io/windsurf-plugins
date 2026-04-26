---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Asimov Atlas**

An interactive, bilingual (EN/CZ) website that transforms Isaac Asimov's Robots/Empire/Foundation reading order flowchart into an animated, explorable timeline. Visitors discover each book through series-themed detail cards with synopses, ratings, cover art, and purchase links. Built with Next.js 15 SSG, GSAP, and tsParticles, hosted on Cloudflare Pages.

**Core Value:** The interactive timeline IS the product — if it doesn't feel amazing to explore, nothing else matters.

### Constraints

- **Tech stack**: Next.js 15 (App Router, SSG), TypeScript, Tailwind CSS v4, shadcn/ui, Magic UI + Aceternity UI, GSAP 3 + ScrollTrigger, Framer Motion 11, tsParticles 3, next-intl, Zustand 5
- **Hosting**: Cloudflare Pages (edge CDN, free tier, auto-deploy via `@cloudflare/next-on-pages`)
- **Performance**: LCP <2.5s mobile, INP <200ms, animation >55fps mobile, JS bundle <150KB gzipped, Lighthouse >95
- **Data**: Static TypeScript files compiled at build time — no database, no API runtime dependency
- **Images**: Cover art from Open Library API at build time, fallback to Cloudflare R2; fair use monitored
- **Analytics**: Cloudflare Web Analytics (privacy-first, no cookie banner)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Critical Discovery: Deployment Path
| Path | Approach | Complexity | Trade-offs |
|------|----------|------------|------------|
| **A: Pure Static Export** | `output: 'export'` in next.config, deploy HTML/CSS/JS to Cloudflare Pages | Low | Simplest; loses ISR, middleware, server actions. Fine for this project since all data is static. |
| **B: OpenNext + Workers** | `@opennextjs/cloudflare` adapter, deploy to Cloudflare Workers | Medium | Full Next.js feature set; more complex build pipeline; Workers has compute limits. |
## Recommended Stack
### Core Framework
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Next.js | 15.5.x | Framework, SSG, routing | Stable, well-tested on Cloudflare Pages static export. v16 exists but v15 is battle-tested for static export and has broader ecosystem compatibility. | HIGH |
| React | 19.x | UI library | Ships with Next.js 15; concurrent features, compiler optimizations | HIGH |
| TypeScript | 5.7+ | Type safety | Non-negotiable for a data-heavy project with 40+ book objects | HIGH |
### Styling
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Tailwind CSS | 4.x | Utility-first CSS | v4 is stable (released Jan 2025), CSS-first config with @theme, 5x faster builds. Series-themed CSS custom properties map perfectly to Tailwind v4's @theme directive. | HIGH |
| CSS Custom Properties | -- | Series theming | Robot=steel-blue, Empire=gold, Foundation=purple via CSS vars. Zero-JS theme switching. | HIGH |
### UI Components
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| shadcn/ui | CLI v4 | Base components (dialog, command palette, cards) | Copy-paste components, not a dependency. Works with Tailwind v4 and Radix primitives. The Cmd+K command palette comes from shadcn's `<Command>` component (built on cmdk). | HIGH |
| Aceternity UI | latest | Timeline visual effects, glowing cards, tracing beams | Copy-paste animated components built on Framer Motion + Tailwind. Perfect for the series-themed book cards and cross-series connection visualizations. | MEDIUM |
| Magic UI | latest | Hero section, star-field overlays, marketing polish | Complements shadcn for landing/marketing elements. "Series A aesthetic" without custom animation work. | MEDIUM |
### Animation
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| GSAP | 3.14.x | Timeline animation, scroll-driven layouts, reading-order re-layout | Industry-standard animation. Now 100% free (Webflow acquired GreenSock, open-sourced everything including ScrollTrigger, SplitText, MorphSVG). ~25KB gzipped. | HIGH |
| GSAP ScrollTrigger | (bundled) | Scroll-driven animation triggers | Bundled with GSAP. Controls era-specific color shifts, timeline node reveals on scroll. | HIGH |
| Motion (formerly Framer Motion) | 12.x | Component enter/exit, layout animations, gesture support | Renamed from `framer-motion` to `motion`. Import from `motion/react`. Use for React-native animations (mount/unmount, layout shifts, drag). GSAP handles the timeline; Motion handles component transitions. | HIGH |
- **GSAP:** Timeline scroll animation, ScrollTrigger-driven reveals, reading-order-to-chronological re-layout animation, star-field color transitions
- **Motion:** Card enter/exit, hover previews, language toggle transitions, command palette open/close, layout animations via `layoutId`
### Particles
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| @tsparticles/react | 3.0.0 | React wrapper for tsParticles | Official React component for tsParticles v3. | MEDIUM |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eletrixa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
