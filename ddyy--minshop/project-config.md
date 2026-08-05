---
trigger: always_on
description: A guide for contributors and automated tooling making changes to minshop: a
---

# AGENTS.md — working on minshop

A guide for contributors and automated tooling making changes to minshop: a
small, full-Cloudflare ecommerce store (Astro 7 SSR on Workers + D1 + R2 +
Stripe / Lightning). Read this before editing. It's the map, the rules, the
recipes, and the traps.

## The loop (run this constantly)

```sh
nvm use 22         # REQUIRED — the supported toolchain runs on Node 22
npm run verify     # complete storefront + D1 + MCP green/red gate
```

`npm run verify` is the single signal that a change is sound: it runs unit tests,
full Astro diagnostics, the production build, the clean-room D1 integration, and
the MCP typecheck/deployment dry run. If it's green, the change holds together.
Run it after every meaningful edit, not just at the end.

Other commands: `npm run dev` (astro dev), `npm run preview` (wrangler dev =
production mode, for testing middleware/auth), `npm run db:migrate` (local D1),
`npm test`.

## Customizing a cloned shop

Use the narrowest customization surface that fits the change:

1. **Admin → Settings** for runtime settings: store identity, time zone,
   storefront toggles, payment rails, email, bot protection, and search. These
   values live in D1 and apply without a deploy.
2. **`src/store.config.ts`** for versioned, build-time settings: currency,
   shipping zones and rates, image dimensions, order numbering, and
   template-only features. Override only changed keys; never customize
   `src/config.ts` defaults directly.
3. **`src/styles/theme.css`** for brand colors, fonts, surfaces, and corner
   radii. Keep structural styles in `global.css` and avoid restyling individual
   components when a shared token will do.
4. **`public/favicon.svg`** for the browser icon. Products, categories, images,
   and other catalog content belong in the admin, not source files.
5. **`wrangler.jsonc`** for optional infrastructure bindings such as image
   optimization, semantic search, and email delivery. Keep optional bindings
   disabled unless the corresponding feature is configured.

Provider credentials belong in Admin → Settings, where the secret vault stores
them encrypted. Deployment-only secrets belong in the platform secret store.
Never put real credentials, customer data, payment data, personal paths, or
local environment files in the repository.

Before handing off a customized shop, run `npm run verify` and review
`git diff --check` plus `git status --short`. Add a new numbered migration for
schema changes; never rewrite a migration that may already have run.

## Architecture in one screen

**Feature folders (vertical slices).** Each owns its data access, types, and
components. Deleting a folder still builds.

```
src/
  config.ts              SCHEMA + DEFAULTS (upstream-owned). getConfig() = source of truth.
  store.config.ts        build-time shop overrides, deep-merged on top
  styles/theme.css       clone-safe brand tokens
  middleware.ts          admin auth gate (fail-closed)
  env.d.ts               Cloudflare.Env binding/secret types
  layouts/               Layout.astro (storefront), AdminLayout.astro
  features/
    products/  db · form · image · ProductForm.astro · sort · search · stock · slug
    orders/    db · number · reservations (atomic checkout stock holds)
    payments/  provider (port) · stripe · opennode · lightning-provider · index (factory)
               lightning/  backend (port) · phoenixd · lnbits · index · rate · pending
    shipping/  calculator (zones + ShippingCalculator port)
    storage/   provider (port) · r2 · index (factory)
    email/     provider (port) · resend · cloudflare · index (factory) · orderConfirmation
    auth/      access (CF Access JWT) · session (admin login cookie) · turnstile · Turnstile.astro
    catalog/   serialize · http  (public API shapes for /api/products)
    cart/ categories/ customers/
  pages/
    index, product/[slug], category/, search, cart, checkout (Lightning own-checkout)
    pay/[publicId] (Lightning invoice page), order/[token] (confirmation)
    admin/ (CRUD UI, login, logout)
    api/ (cart, webhook, admin/*; checkout — form OR JSON {items} → checkout_url;
          products, products/[slug] — public machine-readable catalog)
    images/[...key] (serves R2), sitemap.xml, robots.txt
```

**Ports & adapters (the seams).** Routes depend on interfaces, never on a vendor.
To swap/add a provider, write one adapter file + wire the factory:

| Port | Where | Factory | Adapters |
|---|---|---|---|
| `PaymentProvider` | `payments/provider.ts` | `payments/index.ts` | stripe, opennode, lightning |
| `LightningBackend` | `payments/lightning/backend.ts` | `payments/lightning/index.ts` | phoenixd, lnbits |
| `ShippingCalculator` | `shipping/calculator.ts` | (config-rates) | carrier rates (future) |
| `StorageProvider` | `storage/provider.ts` | `storage/index.ts` | r2 |
| `EmailProvider` | `email/provider.ts` | `email/index.ts` | resend, cloudflare |
| `SearchProvider` | `search/provider.ts` | `search/index.ts` | fts, vector (Workers AI + Vectorize) |

**Bindings.** Access Cloudflare bindings via `import { env } from 'cloudflare:workers'`
(typed by `Cloudflare.Env` in `env.d.ts`). Never `Astro.locals.runtime.env`
(removed in Astro v6). D1 = `env.DB`, R2 = `env.BUCKET`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddyy/minshop](https://github.com/ddyy/minshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
