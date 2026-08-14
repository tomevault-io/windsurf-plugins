---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Features implemented

Real-estate lead-gen landing page for Sunshine Sky City (Quận 7, HCMC), plus an internal admin module. Status as of the latest commit:

- **Public marketing site** (`src/app/page.tsx`): `Hero` → `ProjectInfo` → `Location` → `Amenities` → `GoldValues` → `ApartmentDesign` → `FloorPlan` → `Pricing`, with a scrollspy `Header`, persistent `Footer`, and floating call/Zalo buttons (`FloatingActions`).
- **Entry popup** (`EntryPopup.tsx`): Radix `Dialog` that auto-opens immediately on mount, gated by `sessionStorage["hasSeenEntryPopup"]` so it only shows once per browser session; embeds the same `LeadForm` (`source="Popup Chào Mừng"`).
- **Lead capture pipeline**: `LeadForm` (react-hook-form + Zod) → `POST /api/leads` → Supabase insert → best-effort Telegram notification → best-effort Meta CAPI `Lead` event → toast feedback. See "Architecture" below for the full data flow.
- **SEO**: full `Metadata` export in `src/app/layout.tsx` (title template, keywords, OpenGraph image, Twitter card, canonical, robots) sourced from `siteConfig.seo`.
- **`/cms` admin module**: Basic Auth-gated (`CMS_USERNAME`/`CMS_PASSWORD`) dashboard with a leads table + Excel export, and a settings form to edit the Meta Pixel ID / CAPI token stored in Supabase. See "CMS module" below.
- **Meta Pixel + Conversions API**: pixel script injected site-wide when configured; CAPI `Lead` event fired server-side with hashed PII after every successful lead insert. See "Meta Pixel + Conversions API" below.
- **Verified end-to-end against a real Supabase project**: form submission → row visible in `leads` table → visible in `/cms` (this surfaced and fixed the fetch-caching bug documented below).
- **Cloned from the `sunshine-sky-city` boilerplate; mounted at `nhungtranreal.site/skycity`** via Next.js `basePath` on its own standalone domain — no gateway/rewrite project involved (unlike the base boilerplate). See "Domain architecture" below — this is the one change most likely to break silently if a future edit adds a raw local image `src` or a bare internal `fetch()` call.
- **White-label boilerplate refactor applied**: `src/config/site.ts` centralizes `layoutOrder` (section render order, consumed by `src/app/page.tsx`'s `sectionComponents` registry) and `images` (every local image path, grouped by section). A future clone only ever needs to touch domain/phone/`layoutOrder`/`images` in that one file plus the files under `public/images/`, never the `.tsx` components. `scripts/clone-project.mjs` (inherited from the base boilerplate) automates copying the repo to a new sibling project.

## Project rules (immutable)

These rules were set at project inception and must be preserved across all future changes:

1. **No placeholder code.** Never write `// add code here`, `// similar to above`, or half-finished implementations. Every file must be complete, working code — regardless of length.
2. **Every form/input must validate with Zod** before submission (see `src/lib/validations.ts`, enforced client-side via `@hookform/resolvers/zod` and again server-side in the API route).
3. **Every API fetch must have `try/catch` with a UI fallback** for network/API failure — never let a request fail silently. See `src/app/api/leads/route.ts` and `LeadForm.tsx`'s error toast as the reference pattern.
4. **All contact info (hotline, Zalo, address, email, nav links) lives only in `src/config/site.ts`.** Never hardcode phone numbers or links inside components — import from `siteConfig`.
5. **Mobile-first, fully responsive.** No layout may break or images distort on phone-sized viewports; CTA buttons must stay large/tappable.
6. **Images live in `public/images/` and are referenced as local paths** (`/images/xxx.jpg`), not external URLs. Prefer real project renders over generic stock/placeholder photos when available — see "Image sourcing" below.
7. **A global `<ErrorBoundary>` wraps the app** (in `src/app/layout.tsx`) — do not remove it or render pages outside it.
8. **Follow standard Next.js App Router file organization**: `src/app`, `src/components`, `src/components/ui` (Radix wrappers), `src/lib`, `src/config`.

### Tech stack constraints

- **Next.js 14 (App Router) + TypeScript strict.** Do not downgrade strictness or eject from the App Router.
- **Tailwind CSS v3 only.** Do not introduce MUI, Ant Design, Chakra UI, or any other component/styling library.
- **Radix UI primitives** (`@radix-ui/react-tabs`, `@radix-ui/react-dialog`, `@radix-ui/react-slider`) wrapped under `src/components/ui/` — build new interactive primitives the same way rather than reaching for a third-party UI kit.
- **lucide-react** for all icons.
- **react-hook-form + Zod** for all forms (see rule 2).
- **Supabase (Postgres)** as the lead-capture datastore; schema lives in `supabase/schema.sql` and must be kept in sync with any change to the `leads` table shape.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lehoainamsrt-lang/nhungskycity](https://github.com/lehoainamsrt-lang/nhungskycity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
