---
trigger: always_on
description: - Company: **Pleeter LLC**
---

# BuckThatDuck — Project Context (buckthatduck.com)

## Business
- Company: **Pleeter LLC**
- Product: Custom 3D-printed horse figurines ("Buck") based on Ford Bronco color themes
- Repo: `tspleeter/bronco-buck` | Local: `/Users/home/bronco-buck/`
- Hosting: AWS Amplify (auto-deploy on push to `main`)
- Amplify URL: `main.d2s7zk4p5fxxak.amplifyapp.com`

## Stack
- **Frontend:** Next.js
- **Database:** DynamoDB (`BroncoBuckBuilds`, `BroncoBuckOrders`)
- **Payments:** Stripe (secret key in SSM at `/bronco-buck/stripe-secret-key`; publishable key hardcoded due to `NEXT_PUBLIC_` SSR build limitations)
- **Email:** AWS SES sending from `orders@buckthatduck.com`
- **DNS:** Route 53 (hosted zone `Z01222682JOH1P1Z7BZ0R`)
- **Auth/IAM:** Compute role `bronco-buck-compute-role` assigned in Amplify App Settings → IAM roles (required for SSR Lambda env var access)

## Current Status (as of June 2026)
- Stripe payment integration ✅
- SES order confirmation emails ✅
- Route 53 custom domain live ✅
- `/policies` page (Returns, Shipping, FAQ) ✅
- `/policies/privacy` — Privacy Policy page (Pleeter LLC, 8 Nelke Ct, Hawthorne NJ 07506; Stripe carve-out for card data; AWS named as host; CCPA/GDPR/NJ + multi-state rights). Markdown source copy in `docs/privacy-policy.md`. Footer "Privacy" link added in `layout.tsx` ✅
- Sitewide footer with Pleeter LLC copyright and "🇺🇸 Proudly made in the USA" ✅
- Dark theme UI with gold accents (Andrew's redesign) ✅
- `/orders` route is password-gated ✅
- Multi-view builder (front/right/back/left) ✅
- Gallery: 11 color grid, fixed mane defaults per color, links to builder with pre-selected color+mane ✅
- Home page: two-column hero — text left, Buck+duck photo right (anchored top-left) ✅
- Hero image: `public/assets/hero-buck-duck.png` (Buck biting rubber duck, black bg removed) ✅
- Free rubber duck SVG icon included with every order — shown in build summary, cart, confirmation email ✅
- Nameplate overlay on front view only ✅
- `/policies/terms` — Terms of Service page (Pleeter LLC; Ford non-affiliation disclaimer; NJ governing law — **confirm state of registration**; references /policies for returns/shipping; markdown source in `docs/terms-of-service.md`). Footer "Terms" link added in `layout.tsx` ✅
- `src/middleware.ts` → `src/proxy.ts` migration (Next 16; exported `proxy` function, stray `"use server"` removed) ✅
- `npm run lint` fixed — `next lint` (removed in Next 16) replaced with `eslint .` + `eslint.config.mjs` flat config (FlatCompat extending next/core-web-vitals + next/typescript); `@eslint/eslintrc` added to devDependencies ✅
- `/orders-login` Enter button fixed — `ActionButton` defaults to `type="button"`; added `type="submit"` (button was a dead click since the page shipped; keyboard Enter had been masking it) ✅
- Inbound email LIVE: `orders@buckthatduck.com` now forwards to Todd's inbox via forwardemail.net (free, DNS-only — MX `mx1/mx2.forwardemail.net` + apex TXT `forward-email=orders:...` in Route 53). Also added SPF (`v=spf1 include:amazonses.com -all`) and DMARC (`p=none`) for SES deliverability. **MX-records outstanding issue: RESOLVED** ✅
- Governing law: Pleeter LLC registration **confirmed New Jersey** — ToS clause is correct as-is ✅

## Configurator — bronco-config.json
- **Product:** Bronco Buck Classic (BB001), base price $24.99
- **Base layer:** `base_bronco.png` (transparent)

### Option Groups
| ID | Group | Options |
|----|-------|---------|
| G1 | Body Color | V1 Ruby Red, V2 Velocity Blue, V3 Shadow Black, V15 Eruption Green, V16 Oxford White, V17 Cyber Orange, V18 Carbonized Gray, V19 Cactus Gray, V20 Desert Sand, V21 Azure Gray, V23 Robin's Egg Blue |
| G2 | Mane Style | V4 Regular (+$0), V5 Punk (+$3, **deactivated** July 2026 pending punk renders) |
| G3 | Mane Color | V6 Black (+$0), V7 White (+$2) — now shown in cart summary (imagery is baked into body renders) |
| G4 | Accessories | V8 Sunglasses (+$4) |
| G5 | Stand Style | V9 Standard (+$0) — **hidden from cart summary until imagery ready** |
| G6 | Stand Color | V26 Match Body (+$0, default, first), V27 Black (+$3), V24 Brown (+$3), V25 Sand (+$3) — all live with overlay renders (July 2026); V10/V11 retired but kept in config so old saved builds price correctly |
| G7 | Nameplate | V22 Buck (+$0, default, pre-selected), V12 None, V13 Custom (+$5) |
| G8 | Packaging | V14 Standard Box (+$0) |

### Nameplate
- V22 "Buck" is the default — pre-selected on every new build, free, shows "BUCK" on the preview
- Custom nameplate (V13) is capped at 12 characters
- Nameplate overlay renders on **front view only**
- Overlay zone: top 76.5%, left 10.5%, width 78.8%, height 21.1% of preview container
- Black background, 3px white border all around
- SVG text: viewBox 200×80, fontSize 68, fontWeight 600, white fill, letterSpacing 4
- Hidden groups (G5 only) excluded from `getBuildSummary()` in `src/lib/summary.ts` (G2 + G3 + G6 unhidden July 2026 — paid selections must be itemized)
- **Defaults gotcha:** `getDefaultBuildState()` selects the FIRST active option in each group — the config `default` field is not read. Order options accordingly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tspleeter/bronco-buck](https://github.com/tspleeter/bronco-buck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
