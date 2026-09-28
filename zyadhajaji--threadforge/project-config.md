---
trigger: always_on
description: This file is the single source of truth for any Claude agent (Claude Code, Claude design tools, or AI assistants) working on this repo. Read it fully before writing or editing anything.
---

# ThreadForge — Claude Context

This file is the single source of truth for any Claude agent (Claude Code, Claude design tools, or AI assistants) working on this repo. Read it fully before writing or editing anything.

---

## What this project is

**ThreadForge** is a niche SaaS platform for clothing brands. It lets them:
- Place designs on photorealistic garment templates using a canvas editor
- Manage their brand kit (logos, colors, fonts) per brand
- Organize mockups into collections / seasonal drops
- Export print-ready or digital assets (PNG, WebP, PDF)
- Subscribe to Pro or Studio plans via Stripe

Target users: small-to-mid clothing brands, streetwear labels, print-on-demand sellers.

---

## Tech stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router) | 14 |
| Language | TypeScript | 5 |
| Styling | Tailwind CSS + shadcn/ui | 3.x |
| Canvas editor | Fabric.js | 5 |
| ORM | Prisma | 5 |
| Database | PostgreSQL | any |
| Auth | Clerk | 5 |
| Storage + CDN | Cloudinary | 2 |
| Payments | Stripe | latest |
| Email | Resend | 3 |
| State (client) | Zustand | 4 |
| Validation | Zod | 3 |
| Deployment | Vercel | — |

---

## Directory structure

```
threadforge/
├── CLAUDE.md                        ← you are here
├── .env.example                     ← all required env vars with descriptions
├── .github/
│   ├── workflows/ci.yml             ← lint + typecheck + build on push/PR
│   └── ISSUE_TEMPLATE/
└── apps/
    └── web/                         ← the Next.js application (primary workspace)
        ├── prisma/
        │   ├── schema.prisma        ← canonical data model, edit this first
        │   └── seed.ts              ← seeds template rows (clothing garments)
        ├── public/
        │   ├── templates/           ← placeholder slot for local garment PNGs
        │   └── og/                  ← OpenGraph images
        └── src/
            ├── app/
            │   ├── (auth)/          ← Clerk sign-in / sign-up pages
            │   ├── (dashboard)/     ← authenticated app shell + all pages
            │   │   ├── layout.tsx   ← sidebar nav, UserButton
            │   │   ├── mockups/     ← main grid of user's mockups
            │   │   ├── brands/      ← brand kit management
            │   │   ├── collections/ ← lookbooks / drops
            │   │   ├── editor/      ← Fabric.js canvas editor per mockup
            │   │   └── settings/    ← account, billing, plan
            │   ├── (marketing)/     ← public landing page, pricing, about
            │   └── api/
            │       ├── mockups/     ← GET list, POST create
            │       ├── brands/      ← GET list, POST create
            │       ├── templates/   ← GET list (no auth required)
            │       ├── upload/      ← POST base64 → Cloudinary
            │       └── webhooks/stripe/ ← Stripe event handler
            ├── components/
            │   ├── ui/              ← shadcn/ui primitives only (Button, Dialog, etc.)
            │   ├── editor/          ← Fabric.js canvas, toolbar, color picker, layers panel
            │   ├── mockup/          ← MockupCard, MockupGrid, MockupViewer
            │   ├── brand/           ← BrandCard, BrandKit, ColorPalette
            │   ├── template/        ← TemplateGallery (filterable), TemplateCard
            │   └── shared/          ← UploadZone, Navbar, Sidebar
            ├── lib/
            │   ├── db.ts            ← singleton Prisma client
            │   ├── cloudinary.ts    ← upload, delete, URL helpers
            │   ├── stripe.ts        ← Stripe client + PLANS constant
            │   ├── validations.ts   ← all Zod schemas (source of truth for shapes)
            │   └── utils.ts         ← cn(), slugify(), formatBytes(), absoluteUrl()
            ├── hooks/               ← useEditor(), useMockup(), useBrand()
            └── types/
                └── index.ts         ← re-exports Prisma types + composite types
```

---

## Data model (Prisma)

Read `apps/web/prisma/schema.prisma` for the full schema. Key relationships:

```
User (Clerk identity)
  ├── Brand[]           — one user → many brands (streetwear labels, etc.)
  │     └── Mockup[]   — one brand → many mockups
  ├── Mockup[]          — mockups can also exist without a brand
  │     ├── Template    — every mockup references one garment template
  │     ├── Export[]    — download history (PNG/WebP/PDF)
  │     └── CollectionMockup[] — join table (many-to-many with Collection)
  ├── Collection[]      — lookbooks / seasonal drops
  └── Subscription      — Stripe subscription (one-to-one)

Template                — seeded rows, not user-created
  ├── category: TemplateCategory enum (TSHIRT, HOODIE, HAT, …)
  ├── imageUrl          — Cloudinary URL of the blank garment photo
  ├── maskUrl           — PNG mask for compositing the design layer
  └── designArea        — JSON { x, y, width, height, rotation } in canvas coords
```

When adding a new model or field: edit `schema.prisma` → run `npx prisma migrate dev --name <slug>` → update `src/types/index.ts` if a composite type is needed.

---

## Auth pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zyadhajaji/threadforge](https://github.com/zyadhajaji/threadforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
