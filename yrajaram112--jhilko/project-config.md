---
trigger: always_on
description: > This file is the **single source of truth** for any developer or AI assistant working on this project.
---

# CLAUDE.md — Jhilko: Nepal Local Gift Delivery Platform
## Technical Architecture, Stack Decisions & Implementation Guide

> This file is the **single source of truth** for any developer or AI assistant working on this project.
> Read this fully before writing a single line of code.
>
> **v3.0 (June 2026) revision:** Stack upgraded to current versions (Next.js 16, NestJS 11, Prisma 7, Node 22 LTS, Tailwind v4, Zod 4). `next-pwa` replaced with Serwist. Google Maps replaced with Baato (Nepal-local, ~70% cheaper). Stripe section corrected — **Stripe is NOT available to Nepal-registered companies**; international payments are now Phase-2 via Khalti's international card processing / bank acquirer. Brand finalized as **Jhilko**.

---

## 1. PROJECT IDENTITY

**Name:** `Jhilko` (झिल्को — "spark/flash" in Nepali) — **final brand name**
**Domains:** jhilko.com.np (primary), jhilko.com (redirect)
**Tagline:** *"Send love anywhere in Nepal."*
**Core Mission:** Let anyone in Nepal (or abroad) send a meaningful gift — cake, flowers, sweets, cosmetics, hampers — to someone in any city or village of Nepal, fulfilled by a vetted local partner in that city. Not shipped from KTM. **Sourced and delivered locally.**

**Key Differentiation from Competitors:**
- KoselXpress / YourKoseli serve **diaspora (USA/UK → KTM)** — we serve **domestic Nepal to Nepal (Dharan → Dang, Butwal → Palpa)**
- Pathao/Foodmandu operate **within-city only** — we go **city-to-city**
- We use **local partners** in every city — no centralized warehouse, ultra-low delivery cost

---

## 2. TECH STACK DECISIONS

### 2.1 Frontend — Next.js 16 (App Router)
**Why Next.js:**
- Superior ecosystem for this use case (next/image for product photos, proxy/middleware for auth, ISR for catalog pages)
- Better SEO out of the box (critical for organic discovery)
- More hire-able in Nepal dev market
- App Router allows server components (fast product catalog rendering)
- Native API routes handle webhooks (payment callbacks) cleanly

```
Framework:       Next.js 16.x (App Router, Turbopack, TypeScript 5.x) + React 19.2.x
Styling:         Tailwind CSS v4 (CSS-first config via @theme, no tailwind.config.js)
Animations:      Motion v12+ (package name "motion", NOT "framer-motion")
UI Components:   shadcn/ui (CLI: npx shadcn@latest) + Aceternity UI
State:           Zustand v5 + TanStack Query v5
Forms:           React Hook Form v7 + Zod v4 (import from "zod", use z.email() etc. top-level validators)
Maps:            Baato Maps (Nepal-local, MapLibre GL JS renderer) — see §13
Icons:           Lucide React
Fonts:           Geist (primary) + Noto Sans Devanagari (Nepali) via next/font
PWA:             Serwist (@serwist/next) — next-pwa is UNMAINTAINED, do not use
Image CDN:       Cloudinary (next/image custom loader)
Lint/Format:     ESLint 9 (flat config) + Prettier
```

**Version gotchas Sonnet/codegen must respect (Next.js 16):**
- `cookies()`, `headers()`, `params`, `searchParams` are **async** — always `await` them
- `middleware.ts` is renamed to `proxy.ts` in Next 16
- Use Turbopack (default) — do not add webpack config
- Server Actions for mutations; Route Handlers only for webhooks/external callbacks
- Node.js >= 20.9 required; we standardize on Node 22 LTS

**Design System — Apple Vision / Glassmorphism:**
```css
/* Core CSS Variables — put in globals.css */
:root {
  /* Brand Colors */
  --brand-primary:    #FF4D6D;   /* Deep rose — emotion, gifting */
  --brand-secondary:  #FFB830;   /* Golden amber — Nepali festivals */
  --brand-accent:     #7B2FBE;   /* Royal purple — premium trust */
  
  /* Surfaces */
  --surface-glass:    rgba(255,255,255,0.08);
  --surface-glass-border: rgba(255,255,255,0.15);
  --surface-card:     rgba(255,255,255,0.72);
  --backdrop-blur:    20px;
  
  /* Text */
  --text-primary:     #0A0A0B;
  --text-secondary:   #6B7280;
  --text-muted:       #9CA3AF;
  
  /* Gradients */
  --gradient-hero:    linear-gradient(135deg, #FF4D6D 0%, #FFB830 50%, #7B2FBE 100%);
  --gradient-card:    linear-gradient(145deg, rgba(255,77,109,0.1) 0%, rgba(123,47,190,0.05) 100%);
  
  /* Spacing tokens */
  --radius-sm:        8px;
  --radius-md:        16px;
  --radius-lg:        24px;
  --radius-xl:        32px;
  --radius-pill:      9999px;
}
```

**Motion Principles:**
- Spring physics everywhere (no `ease-in-out` flat curves)
- Page transitions: slide + fade (300ms spring)
- Cards: `whileHover={{ y: -4, scale: 1.02 }}` with shadow
- Buttons: `whileTap={{ scale: 0.97 }}`
- Modals: scale from 0.94 + fade in (apple-style)
- Skeleton loaders: pulse shimmer animation
- Delivery tracker: animated progress ring + bouncing location pin

---

### 2.2 Backend — NestJS + PostgreSQL + Redis

**Why NestJS:**
- TypeScript-native (matches frontend, single language stack)
- Excellent WebSocket support via `@nestjs/websockets` + Socket.IO
- Built-in support for BullMQ (job queues for delivery status, SMS)
- Modular architecture scales well as features grow
- Nepal developers familiar with it (common in KTM tech scene)

```
Runtime:         Node.js 22 LTS
Framework:       NestJS 11.x (stable — do NOT use v12 alphas until GA)
Database:        PostgreSQL 17 (Neon)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yrajaram112/jhilko](https://github.com/Yrajaram112/jhilko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
