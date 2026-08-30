---
trigger: always_on
description: Pawples (thepawples.com) is an **Indian pet e-commerce platform** with a core subscription box product. The mission: deliver curated, vet-approved treats, toys, and wellness products for Indian pets every month. Tagline: **"Because they're family."**
---

# Pawples — Project Context for Claude

## What is Pawples?
Pawples (thepawples.com) is an **Indian pet e-commerce platform** with a core subscription box product. The mission: deliver curated, vet-approved treats, toys, and wellness products for Indian pets every month. Tagline: **"Because they're family."**

Target market: India (INR pricing, Razorpay payments, pan-India shipping via Shiprocket/Delhivery).
Pet categories: **Dogs, Cats, Birds, Small Animals** (hamsters, rabbits, guinea pigs, etc.).

---

## Tech Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 14 — App Router |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS (custom brand config) |
| Database & Auth | Supabase (Postgres + Supabase Auth) |
| Payments | Razorpay (orders + subscriptions) |
| Email | Resend |
| Image CDN | Cloudinary |
| Fonts | Nunito (headings, 800/900 weight) + DM Sans (body) from Google Fonts |

---

## Brand

```
Primary (Plum):    #5B2D8E
Accent (Gold):     #F5A623
Background (Ivory):#FDF8F2
Text (Deep Bark):  #1A1A2E
```

**Logo mark**: A plum-coloured ellipse (nose shape) with two gold nostril ellipses inside — rendered as an inline SVG in `components/layout/PawplesLogo.tsx`.

Tailwind colour tokens: `plum`, `gold`, `ivory`, `bark` (see `tailwind.config.ts`).
Font family tokens: `font-nunito`, `font-dm-sans`.

---

## Project Structure

```
/
├── app/                        # Next.js App Router pages
│   ├── layout.tsx              # Root layout (fonts, metadata)
│   ├── globals.css             # Tailwind base + global styles
│   ├── page.tsx                # Homepage (fully built)
│   ├── shop/
│   │   ├── page.tsx            # All-products shop
│   │   ├── [category]/         # Category page (dogs/cats/birds/small-animals)
│   │   └── [category]/[slug]/  # Product detail page (PDP)
│   ├── subscribe/
│   │   ├── page.tsx            # Subscription landing
│   │   ├── quiz/               # Pet personalisation quiz
│   │   ├── choose/             # Plan selector (Snoozy/Zoomies/Royale)
│   │   ├── addons/             # Addon picker
│   │   └── success/            # Post-subscription confirmation
│   ├── search/                 # Search results
│   ├── about/ blog/ contact/   # Informational pages
│   ├── login/ signup/          # Auth pages
│   ├── forgot-password/        # Password reset
│   ├── onboarding/             # Post-signup pet setup
│   ├── cart/                   # Cart page
│   ├── checkout/               # Checkout (Razorpay integration)
│   ├── order/[id]/             # Order detail / tracking
│   ├── account/                # Account dashboard
│   │   ├── orders/             # Order history
│   │   ├── subscription/       # Manage subscription
│   │   ├── pets/               # Pet profiles
│   │   ├── addresses/          # Saved addresses
│   │   ├── rewards/            # Reward points / referrals
│   │   └── settings/           # Profile & notification settings
│   ├── admin/                  # Internal admin (protected)
│   │   ├── products/           # Product management
│   │   ├── orders/             # Order management
│   │   ├── subscriptions/      # Subscription management
│   │   ├── customers/          # Customer management
│   │   └── promos/             # Promo code management
│   └── privacy/ terms/ shipping/ returns/  # Legal & policy pages
│
├── components/
│   ├── layout/
│   │   ├── PawplesLogo.tsx     # SVG logo mark + wordmark
│   │   ├── Navbar.tsx          # Sticky nav with mobile drawer
│   │   └── Footer.tsx          # Full footer with social links
│   ├── ui/
│   │   ├── Button.tsx          # btn-primary / secondary / outline / ghost
│   │   └── Badge.tsx           # Colour-coded badges
│   ├── shop/                   # Product cards, filters, gallery
│   ├── subscription/           # Plan cards, quiz steps, addon picker
│   ├── account/                # Account section components
│   └── admin/                  # Admin table, stat cards, etc.
│
├── lib/
│   ├── supabase.ts             # Supabase client (anon) + supabaseAdmin()
│   ├── razorpay.ts             # Razorpay client, createOrder, verifySignature
│   └── utils.ts                # cn(), formatPrice(), slugify(), etc.
│
├── types/
│   └── index.ts                # All TS types: Product, Order, Subscription, Pet, User, CartItem, etc.
│
├── public/
│   └── images/                 # Static assets
│
├── CLAUDE.md                   # ← This file
├── .env.local.example          # Required env vars (copy → .env.local)
├── tailwind.config.ts          # Brand colours + fonts + animations
├── next.config.ts              # Image domains (picsum, cloudinary)
└── tsconfig.json               # Strict TS, path alias @/*
```

---

## Key Types (types/index.ts)

- **Product** — slug, variants (SKU/price/stock), category, rating, status
- **ProductVariant** — id, sku, price (paise), stock, weight, attributes
- **CartItem** — productId, variantId, name, image, price, quantity
- **Order** / **OrderItem** — status lifecycle: pending → confirmed → processing → shipped → out_for_delivery → delivered

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shivaprasad909/Pawples](https://github.com/shivaprasad909/Pawples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
