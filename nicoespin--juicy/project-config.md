---
trigger: always_on
description: This project is the official landing page for **Juicy Hamburgers**, a premium burger brand from **Villa Carlos Paz**, with **2 existing locations** and a **future Buenos Aires launch**.
---

# AGENTS.md — Juicy Hamburgers Landing

## 1. Project Context

This project is the official landing page for **Juicy Hamburgers**, a premium burger brand from **Villa Carlos Paz**, with **2 existing locations** and a **future Buenos Aires launch**.

The site must feel:

- award-worthy
- modern
- fast
- premium
- highly visual
- conversion-oriented

This is **not** a generic restaurant site.
It must communicate **desire**, **brand attitude**, and **place identity**.

Primary slogan:

- **Taste the Difference**

Primary business goals:

- drive traffic to physical locations
- support delivery intent
- capture leads for Buenos Aires waitlist
- strengthen brand perception

---

## 2. Core Stack

- **Next.js** (App Router)
- **TypeScript**
- **Tailwind CSS**
- **GSAP** for premium motion and scroll-based storytelling

Preferred implementation style:

- server-first where possible
- client components only when interactivity/animation requires it
- clean composition
- reusable sections and UI primitives
- strong visual consistency
- minimal dependencies

---

## 3. Product Principles

Every contribution to this codebase must follow these principles:

1. **Brand before decoration**
   - visuals must reinforce Juicy’s identity
   - never add trendy UI just for the sake of it

2. **Motion with intention**
   - animation must support hierarchy, reveal, delight, or conversion
   - avoid noisy, random, or excessive movement

3. **Product-first storytelling**
   - the burgers are the heroes
   - content should make the food feel desirable immediately

4. **Fast by default**
   - performance is part of the design
   - heavy effects must be justified and optimized

5. **Mobile-first experience**
   - most users will likely discover the brand on mobile
   - every section must work beautifully on small screens first

6. **Premium minimalism**
   - bold, memorable, retro-american inspiration
   - but always polished, modern, and restrained

---

## 4. Visual Direction

The brand aesthetic combines:

- retro-american burger culture
- premium editorial presentation
- clean contemporary layout systems
- warm, tactile materials and subtle texture

### Brand Colors

- juicy-red: `#C41E1E`
- juicy-red-dark: `#8B1010`
- juicy-red-light: `#E84040`
- juicy-cream: `#FAF7F0`
- juicy-cream-dark: `#F0EBE0`
- juicy-black: `#1A1008`
- juicy-gray: `#8A8070`
- juicy-white: `#FFFFFF`

### Typography

- Display / logo vibe: `Pacifico` or `Satisfy`
- Headlines: `Bebas Neue`
- Body/UI: `DM Sans`
- Accent phrases: `Playfair Display Italic`

### Visual Motifs

- red/white checkerboard
- subtle grain
- kraft/paper-inspired warmth
- bold food photography
- red illustrated dog mascot (“Juicy Dog”)

---

## 5. Section Strategy

The landing is composed around these sections:

1. Hero
2. Philosophy
3. Menu Showcase
4. Immersive Gallery / The Vibe
5. Locations + Buenos Aires Waitlist
6. Social Proof / Reviews
7. Footer

When building or editing sections:

- preserve narrative flow
- maintain strong contrast between sections
- avoid repetitive layouts
- each section must have a clear visual idea

---

## 6. Motion Rules (GSAP)

GSAP is a core part of the experience, but it must follow strict rules:

### Use GSAP for:

- hero entrances
- staggered reveals
- scroll-triggered parallax
- marquee motion
- mascot reactions
- hover interactions on food/product cards
- clip-path reveals when they clearly improve perceived quality

### Avoid:

- animating everything on screen at once
- long blocking intro animations
- motion that delays access to content
- excessive bouncing or cartoon-like easing unless it fits the mascot

### Motion style

- confident
- premium
- playful in controlled amounts
- smooth, high-end, tactile

### Technical rules

- initialize GSAP only on client
- always clean up contexts/timelines
- use `prefers-reduced-motion`
- avoid layout thrashing
- use transforms and opacity first
- scroll-based animations must feel performant on mid-range mobile devices

---

## 7. Performance Rules

Performance is a product requirement.

Always prefer:

- `next/image`
- `next/font`
- lazy loading where appropriate
- optimized image sizes and formats
- minimal client-side JS
- section-level code separation if justified
- transform/opacity animations over layout-changing animations

Avoid:

- oversized hero images without optimization
- unnecessary animation libraries beyond GSAP
- large third-party widgets unless clearly valuable
- shipping decorative code with no business or UX value

---

## 8. Accessibility Rules

Accessibility is mandatory, not optional.

Always ensure:

- semantic HTML
- visible focus states
- keyboard navigability
- sufficient contrast
- descriptive alt text
- accessible forms
- reduced-motion handling
- links and buttons are clearly distinguishable

No visual change should break:

- keyboard navigation
- focus visibility
- reading order
- screen reader understanding

---

## 9. SEO & Content Rules

This is a local-business website, so SEO must support discovery.

Priorities:

- strong metadata per page
- local relevance
- structured, readable headings
- location-based copy
- optimized Open Graph

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NicoEspin/Juicy](https://github.com/NicoEspin/Juicy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
