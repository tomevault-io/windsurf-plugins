---
trigger: always_on
description: Marketing site for **Extreme Clean Auto Detailing** — a mobile detailing business owned by **Blake**, serving Macomb and Oakland counties in Michigan. Premium positioning ("#1 in the area"), phone-call-first conversion, no pricing shown on site.
---

# CLAUDE.md

Marketing site for **Extreme Clean Auto Detailing** — a mobile detailing business owned by **Blake**, serving Macomb and Oakland counties in Michigan. Premium positioning ("#1 in the area"), phone-call-first conversion, no pricing shown on site.

Stack: Vite + React 18 + TypeScript + Tailwind 3 + `lucide-react` icons. Deploys to Vercel. Current state is a single-page scroll scaffolded by Bolt; actively being rebuilt into a multi-page site.

---

## Known failure modes — do not repeat

**1. Color extraction from reference images.** Claude cannot reliably extract exact hex values from PNG/JPG screenshots. Approximated colors are close but wrong, which breaks brand consistency. When working from visual references: use reference images for *composition and layout only*, never for color picking. Always use the defined brand hex values. If a color not in the palette is genuinely needed, propose the exact hex to Blake/Gabriel for confirmation before using it.

**2. Service area map — use static image + SVG overlay only.** The service area map is a static PNG screenshot saved at `public/projects/service-areas-reference-dark-map-overlays-2026-04-15.png`, displayed at fixed dimensions in the ServiceAreas component, with a freehand SVG `<path>` overlay tracing Blake's coverage area. The outline stroke uses brand.yellow (`#FFC900`) via the `stroke-brand-yellow` Tailwind utility; the interior fill uses brand.yellow at 15% opacity (`fill-brand-yellow` with `fill-opacity="0.15"`) to subtly highlight the served region. Stroke width 5px, rounded linejoins and linecaps. Do NOT use Google Maps JS API, Mapbox, Leaflet, OpenStreetMap tile libraries, or any other map service at runtime. Do NOT revert to a custom hand-drawn SVG of Michigan counties. The map image is a one-time export — if Blake's service area changes, replace the PNG and update the SVG path coordinates. A reference image at `public/projects/service-areas-reference-hand-drawn-boundary-2026-04-22.png` shows the intended outline shape; it is used for deriving path coordinates only and is NOT loaded at runtime.

**3. Copy changes require confirmation.** All existing site copy was pulled from Blake's live Wix site (extremecleanauto.com) and represents his approved voice. Do not rewrite, "improve," or SEO-optimize copy unprompted. See Copy Rules below.

**4. Uninvited design changes.** Don't rearrange sections, swap animations, or restructure components unprompted. Propose-then-implement. See Design Rules below.

---

## Architecture

**Target structure (in progress, not yet built):**

- `/` — Home (teaser sections: About, Service Areas, Reviews, Services, Ceramic Coatings, Contact)
- `/detailing` — Full mobile detailing services page (3 services)
- `/ceramic-coatings` — Full ceramic coatings page (3 packages — content TBD from Blake)
- `/contact` — Inquiry form page (submits to Fieldd CRM)

Current codebase is single-page (`App.tsx` renders all sections sequentially). Routing migration to `react-router-dom` is pending. When splitting sections into pages, ask before refactoring existing components — some may need to serve as both homepage teasers and full-page versions.

**Lead capture flow:** Website inquiry form → `/api/submit-lead.ts` Vercel serverless function → Fieldd CRM webhook → Fieldd's automations handle SMS/email follow-up → Blake manually books appointments inside Fieldd. The site is a lead capture tool, not a booking tool. Blake does not want clients booking their own appointments.

**Supabase is installed but unused.** The `@supabase/supabase-js` dependency is a leftover from the Bolt scaffold. It can be removed — the project uses Fieldd for CRM, not Supabase.

---

## Environment variables

All server-side secrets (no `VITE_` prefix) live in Vercel env settings and locally in `.env.local` (gitignored). Never commit secrets.

- `FIELDD_WEBHOOK_URL` — endpoint for posting leads to Fieldd (server-side). **TBD — Blake's Fieldd integration path is not yet determined.** Build the inquiry form with a clearly-marked `submitLead()` TODO that currently logs to console or shows a success toast; replace with real Fieldd integration once CRM access is available.
- `GOOGLE_PLACES_API_KEY` — for pulling live Google reviews via `/api/google-reviews.ts` (server-side). Not yet obtained.
No other env vars currently planned. Spam protection (Turnstile/hCaptcha) not implemented — can add later if spam becomes a problem. Email notifications to Blake on form submit are handled by Fieldd's automations, not by the site.

---

## Brand system

Palette:

- **Blue:** `#1E90FF` — primary accent, CTAs, highlights (existing hex literal convention)
- **Blue dark:** `#0055cc` — gradient partner to blue (existing hex literal convention)
- **Dark:** `#080808` — near-black background (existing hex literal convention)
- **Yellow:** `#FFC900` — new accent, used strategically (NOT everywhere — rare, high-attention). Defined as `brand.yellow` in `tailwind.config.js`.

**Color usage rules:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gbq1019/extreme-clean-auto-detailing](https://github.com/gbq1019/extreme-clean-auto-detailing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
