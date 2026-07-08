---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**A2C Logistics CO. — Driver Recruiting Site**

A full rebuild of `a2clogistics.com` — a driver-recruiting site for **A2C Logistics CO.**, a Lincoln, NE trucking company. The site is built for **two audiences only — owner-operators (lease-on) and company drivers (W2)** — and exists to convert visiting drivers into qualified leads for A2C's recruiting team.

The current Vite/React site mixes shipper messaging (Services, Fleet) with driver messaging across six generic pages. The new site drops shipper content entirely, leads with a story-driven *"driver-first"* identity, and routes every visitor toward a single quick-apply form. A dedicated section introduces the four sister brands in the A2C ecosystem (LTTR, LTS, DP, OTTS), reinforcing the *"you're joining an ecosystem, not just a fleet"* differentiator.

**Core Value:** **Every visiting driver — owner-op or company — leaves the site having either submitted the quick-apply form, or knowing exactly who A2C is and why "Driven to be different" is more than a tagline.** Conversion is the bar; trust is the moat.

### Constraints

- **Tech stack**: Open — no strong preference. SEO is critical (driver-job search queries), so the working assumption is **Next.js (App Router) + Tailwind 4 + MDX** unless the planning phase surfaces a better fit. Astro is the alternative if interactivity stays minimal. Existing `package.json` deps (framer-motion, lucide-react, base-ui, shadcn) carry over where useful.
- **Content**: All copy, pay numbers, testimonials, and structured data live in MDX/markdown in the repo (no headless CMS). Edits are made via PR or a git-based CMS UI like TinaCMS.
- **Hosting**: Netlify or Cloudflare Pages (final pick deferred to deployment phase). Both support Next.js SSG/SSR and Edge Functions for the form handler.
- **Form delivery**: Submissions email recruiting + write a row to a Google Sheet or Airtable. Built behind a swappable adapter so a future ATS (Tenstreet/DriverReach) can replace the email/sheet sink without rebuilding the form UI.
- **Brand fidelity**: Nevis Bold typography is licensed — need a licensed copy or web-font equivalent before launch. Color palette and logo system are non-negotiable per the brand book.
- **Compliance**: DOT/FMCSA recruiting copy norms apply (no protected-class questions, voluntary consent for recruiter contact). Privacy policy required because of PII collection.
- **Performance**: Mobile-first; LCP < 2.5s on 4G; no layout shift on hero or testimonial swaps.
- **Accessibility**: WCAG AA minimum.
- **Single-site scope**: This project is A2C Logistics CO. only. Sister brands are referenced and linked, never absorbed.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## TL;DR — Recommended Stack
| Layer | Pick | Confidence | Why for *this* project |
|---|---|---|---|
| Framework | **Astro 6 (`astro@6.2.2`)** | HIGH | SEO-first, ships ~0 JS by default, native MDX content collections with Zod schemas, native `Image` + font optimization, Cloudflare/Netlify adapters are first-party. The two interactive needs (form, OO/Company toggle) are textbook Astro Islands. |
| Styling | **Tailwind CSS v4 (`tailwindcss@4.2.4`) via `@tailwindcss/vite`** | HIGH | Already the user's choice, v4 stable, Astro/Vite plugin is the canonical install. CSS-first config = no `tailwind.config.js`. |
| Component primitives | **shadcn/ui CLI v4 (`shadcn@4.6.0`) — Astro template** | HIGH | The shadcn CLI v4 (March 2026) added a first-class Astro template. Same components/registry the user already knows from React. |
| Content | **Astro Content Collections + `@astrojs/mdx@4.4.3`** with **Zod schemas** | HIGH | Frontmatter-typed pay numbers, testimonials, brand data. No second toolchain (Velite/Fumadocs unnecessary). |
| Form library | **Conform (`@conform-to/zod` + `@conform-to/react`)** | HIGH | Progressive enhancement (works without JS), Zod-shared schema between Astro Action handler and the React island. Same Zod schema validates client + server. |
| Form transport | **Astro Actions** (server endpoint at `/_actions`) | HIGH | Type-safe, runs on the host's serverless runtime, Astro's native pattern in v5+. |
| Form sinks | **Resend (`resend@6.12.2`) for email + Google Sheets API or Airtable REST** behind a small adapter | HIGH | Resend is the modern default for transactional email; the adapter satisfies REQ-FUNNEL-02 + the "swappable so an ATS can replace it" constraint. |
| Spam protection | **Cloudflare Turnstile (`@marsidev/react-turnstile@1.5.1`) + honeypot field** | HIGH | Free, privacy-friendly, no user-visible CAPTCHA, works on either host. |
| Hosting | **Cloudflare Pages** (primary recommendation) — Netlify is a defensible second choice | MEDIUM | See "Host Decision" section below. Cloudflare wins on free-tier function invocations and global edge; Netlify wins on DX polish and built-in form/email plumbing. |
| Fonts | **`next/font/local`-equivalent in Astro: `astro:assets` Fonts API + self-hosted licensed Nevis Bold + Fontsource for Avenir-alike** | HIGH | Foundry-license fonts must be self-hosted (no CDN). Astro's Fonts API (stable in 6.x) handles `font-display: swap`, preload, subsetting. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alxmara1405/a2c-logistics-website](https://github.com/alxmara1405/a2c-logistics-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
